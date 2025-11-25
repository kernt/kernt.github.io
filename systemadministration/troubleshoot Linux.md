# Case Preparation

Let’s run the following docker container on a Linux host to simulate some high disk I/O scenario:

```
$ docker run --name=app -p 80:80 -itd tonylixu/word-pop  
9bc4614419bc606064b3d35c5ba86be3fb55617dcd0e61791b4326f6df7c8c37  
```
  
```
$ docker ps  
CONTAINER ID   IMAGE             COMMAND            CREATED         STATUS         PORTS                               NAMES  
9bc4614419bc   feisky/word-pop   "python /app.py"   6 minutes ago   Up 6 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp   app
```

Once the container is up and running, let’s visit the url “http://127.0.0.1” using `curl` command:

```
$ curl http://127.0.0.1  
hello world
```

We can see that the case starts normally. Now let’s query a specific endpoint:

---
$ time curl http://127.0.0.1/popularity/word  
{  
  "popularity": 0.0,  
  "word": "word"  
}real 0m44.134s  
user 0m0.000s  
sys 0m0.008s


From the above result, you can see that it took `44` seconds to get the result back! What’s going on? Why the query takes so long?

# Troubleshooting Steps

You might think it has something to do with networking, but before we dive any deeper, let’s use `top` command to observe the system performance:

![](https://miro.medium.com/v2/resize:fit:700/0*vDsQBBH79_HqKg18.png)

Looking at the output of top, you can see that the iowait of both CPUs is very high. Especially for CPU1, iowait has reached 80%, and there are 2.2GB of remaining memory, which seems to be sufficient.

Looking further down, the CPU usage of a python process with PID 3920 in the process section is slightly higher, reaching 28%. Although 28% is not a performance bottleneck, it is a bit suspicious — it may be related to the increase in `iowait`.

Let’s do a `ps` command:

```
$ ps aux | grep 3920  
root      3920 13.1  0.7 103860 29232 pts/0    Sl+  15:14   2:46 /usr/local/bin/python /app.py
```

From the output of ps, you can see that this process with high CPU usage is exactly our case application. But don’t rush to analyze the CPU problem yet, because CPU usage is not critical high at this point, the `iowait` has reached 90% is our first solution.

Next, let’s continue our I/O analysis by using `iostat` command:

- `-d`: I/O performance indicator
- `-x` : Display extended stats

![](https://miro.medium.com/v2/resize:fit:700/0*gDtJ8YKyF1lfLlFp.png)

From the output of `iostat` , we observed that `wKB/s` is quite high, **62MB/second**, and the I/O usage of disk `xvda` is 100%. Obviously, we are experiencing a I/O saturation. Moreover, the averaged queue length of write requests that were issued is **34.85**, also a high number.

> **So now the question is, how do we know which process is causing the I/O issue? We have a suspicious Python process with PID 3920, but how do we make sure the Python process is the I/O killer?**

Let’s use `pidstat` to check the process:

![](https://miro.medium.com/v2/resize:fit:700/0*qp9NPLs3tIVeA_fy.png)

From the above output, we can see that it is the 3920 Python process that causing the I/O performance bottleneck.

At this point, you probably think that the next step is very simple. We can just use `strace` to confirm whether it is writing a file, and then use `lsof` to find the file corresponding to the file descriptor. Then we fix our code and redeploy the application.

Let’s try it:

```
$ strace -p 3920 2>&1 | grep write  
...hangs
```

Sadly, there is no output related to any `write` system calls. At this point, you may ask, what is going on? Why there are no `write` system calls at all? If there are no `write` system calls, how come this process 3920 is causing a I/O issue? Isn’t this strange?

For file writing, there should be a corresponding write system call, but no trace can be found with the existing tools. At this time, it is time to think about the problem of changing tools. How can I know where the file is being written?

Let’s use `filetop` ([https://github.com/iovisor/bcc/blob/master/tools/filetop.py](https://github.com/iovisor/bcc/blob/master/tools/filetop.py)), a new tool for tracing reading and writing files in the kernel. You can find the install instructions in the link I just provided.

Once installed, let’s run `filetop`

```
$ filetop -C  
15:57:06 loadavg: 2.22 2.20 1.77 1/193 21777TID    COMM             READS  WRITES R_Kb    W_Kb    T FILE  
21776  python           0      1      0       4882    R 594.txt  
21776  python           0      1      0       4736    R 583.txt  
21776  python           0      1      0       4589    R 582.txt  
21776  python           0      1      0       4394    R 580.txt  
21776  python           0      1      0       3857    R 576.txt  
21776  python           0      1      0       3808    R 592.txt  
21776  python           0      1      0       3271    R 579.txt  
21776  python           0      1      0       2978    R 593.txt  
21776  python           0      1      0       2929    R 575.txt  
21776  python           0      1      0       2929    R 573.txt  
21776  python           0      1      0       2880    R 577.txt  
21776  python           0      1      0       2832    R 586.txt  
21776  python           0      1      0       2587    R 574.txt  
21776  python           0      1      0       2441    R 578.txt  
21776  python           0      1      0       2441    R 587.txt  
21776  python           0      1      0       2392    R 585.txt  
21776  python           0      1      0       2343    R 590.txt  
21776  python           0      1      0       2294    R 591.txt  
21776  python           0      1      0       2246    R 572.txt  
21776  python           0      1      0       2246    R 584.txt  
```
....

You will see that `filetop` outputs 8 columns, which are thread ID, thread command line, number of reads and writes, size of reading and write (in KB), file type, and file name of reading and write.

After observing for a while, you will find that every once in a while, the python application with thread number 21776 will first write a large number of txt files, and then read a lot.

Which process does the thread with thread number 21776 belong to? We can check it with the `ps` command:

```
$ ps -efT | grep 21776  
root      3920 21776  3888 30 15:59 pts/0    00:00:07 /usr/local/bin/python /app.py
```

We see that this thread is exactly the thread of case application 3920. Now how do we know which file(s) that it is writing?

We can use the `opensnoop` ([https://github.com/iovisor/bcc/blob/master/tools/opensnoop.py](https://github.com/iovisor/bcc/blob/master/tools/opensnoop.py)) command:

```
PID    COMM               FD ERR PATH  
3920   python              6   0 /tmp/2ec58410-919d-11ec-aa6a-0242ac110002/657.txt  
3920   python              6   0 /tmp/2ec58410-919d-11ec-aa6a-0242ac110002/658.txt  
3920   python              6   0 /tmp/2ec58410-919d-11ec-aa6a-0242ac110002/659.txt  
3920   python              6   0 /tmp/2ec58410-919d-11ec-aa6a-0242ac110002/660.txt  
3920   python              6   0 /tmp/2ec58410-919d-11ec-aa6a-0242ac110002/661.txt  
3920   python              6   0 /tmp/2ec58410-919d-11ec-aa6a-  
...  
3920   python              6   0 /tmp/2ec58410-919d-11ec-aa6a-0242ac110002/667.txt  
3920   python              6   0 /tmp/2ec58410-919d-11ec-aa6a-0242ac110002/668.txt  
3920   python              6   0 /tmp/2ec58410-919d-11ec-aa6a-0242ac110002/669.txt  
3920   python              6   0 /tmp/2ec58410-919d-11ec-aa6a-0242ac110002/670.txt

```
This time, through the output of `opensnoop`, we can basically judge that the python application will dynamically generate a batch of files to temporarily store data, and delete them when they are used up, resulting in very slow processing.

Let’s check the content of `ppy.py` :

```
$ docker cp 9bc4614419bc:app.py ./  
$ cat app.y
```

```
@app.route("/popularity/<word>")   
def word_popularity(word):   
  dir_path = '/tmp/{}'.format(uuid.uuid1())   
  count = 0   
  sample_size = 1000   
     
  def save_to_file(file_name, content):   
    with open(file_name, 'w') as f:   
    f.write(content)try:   
    # initial directory firstly   
    os.mkdir(dir_path)# save article to files   
    for i in range(sample_size):   
        file_name = '{}/{}.txt'.format(dir_path, i)   
        article = generate_article()   
        save_to_file(file_name, article)# count word popularity   
    for root, dirs, files in os.walk(dir_path):   
        for file_name in files:   
            with open('{}/{}'.format(dir_path, file_name)) as f:   
                if validate(word, f.read()):   
                    count += 1   
    finally:   
        # clean files   
        shutil.rmtree(dir_path, ignore_errors=True)return jsonify({'popularity': count / sample_size * 100, 'word': word})
```

As you can see from the source code, in this case application, during the processing of each request, a batch of temporary files will be generated, then read into the memory for processing, and finally the entire directory will be deleted.

# Conclusion

In this article, we troubleshooted a case of high I/O latency. First, we analyzed the CPU and disk usage of the system with `top` and `iostat`. We found the disk I/O bottleneck, and also knew that the bottleneck was caused by the case application.

Next, we used `strace` to observe the system calls, and `filetop` + `opensnoop` command to identify the thread and files that being written into.