> **Table of contents**  
> - [Why do you need a password manager](https://medium.com/@p.thapar99/pass-the-standard-unix-password-manager-a42083698ddf#1d83)  
> - [How to use a password manager](https://medium.com/@p.thapar99/pass-the-standard-unix-password-manager-a42083698ddf#69cd)  
> - [Setting up Pass](https://medium.com/@p.thapar99/pass-the-standard-unix-password-manager-a42083698ddf#5f83)  
> - [Setting up a GPG key](https://medium.com/@p.thapar99/pass-the-standard-unix-password-manager-a42083698ddf#e3c1)  
> - [Inserting passwords](https://medium.com/@p.thapar99/pass-the-standard-unix-password-manager-a42083698ddf#54f6)  
> - [Generating password](https://medium.com/@p.thapar99/pass-the-standard-unix-password-manager-a42083698ddf#a6c2)  
> - [Retrieving password](https://medium.com/@p.thapar99/pass-the-standard-unix-password-manager-a42083698ddf#0acb)  
> - [Removing password](https://medium.com/@p.thapar99/pass-the-standard-unix-password-manager-a42083698ddf#ab90)  
> - [Listing the passwords](https://medium.com/@p.thapar99/pass-the-standard-unix-password-manager-a42083698ddf#f6ee)  
> - [Further reading](https://medium.com/@p.thapar99/pass-the-standard-unix-password-manager-a42083698ddf#0fcd)

# Why do you need a password manager?

Everyone, in fact, should have one. If you’re still on the fence, consider the following advantages of using a password manager.

- You don’t have to write passwords over and over again
- You don’t have to remember them
- You may create unique passwords and avoid ‘credential stuffing’
- All of your passwords can be generated and stored in one place.

# How to use a password manager?

Using a password manager with a graphical user interface, such as bitwarden or KeePass, is simple.
But if you’d rather use a command-line tool, I’ve got you covered.
I’ll walk you through every step of using Pass, a CLI-based password manager, in this article.
So, to begin, go to the official website and learn why you should use Pass!
Now, let's get started without further ado by downloading Pass.
If you're using an arch-based distribution, go ahead and type `sudo pacman -S pass` and you should get a similar output
If you type `pass` again, it would instruct you to **initialize a password store**
So, let’s do it! The command is `pass init <ID of your GPG key>`
If you haven’t set a GPG key follow along, else you can skip this step

# Setting up a GPG key

type `gpg --gen-key` to generate the key.

- You’ll be prompted to type **real name**. This name will be used while initializing the password store.
- Then set your email.
- A prompt similar to the one below will pop up and ask you for Passphrase, which is kind of like a master password. Make sure to choose a strong one. Once you are happy with it go ahead and click OK.

![](IE96aKnxgfm-nQBa.webp)

With this, you got a GPG key.

# Inserting passwords

If you read the documentation, you’ll notice that **Pass keeps passwords in encrypted files that are separate.**

As a result, we should name the files according to their intended use.
Your GitHub account password, for example, may be saved in a file titled Github.com.
You may even organise your password files by putting them in different folders.
For example, if you want to make a file test in the personal folder, type:  
pass insert personal/test

In general, the command is `pass insert <path>`

When you type the command and hit enter, you’ll be prompted for the password and to retype it.

![](E58aG98hhlj-cRFY.webp)

This is how easily you can store the password.

# Generating a password

Yes, you don’t have to be creative to think of a unique password every time. Let the password manager generate one for you.

Ask it,`pass generate <path>` .

![](kg5D3ucI08wtcMcnoJzApg.webp)

If there’s someone around you and you want them to eavesdrop on your password, you can directly copy it to the clipboard with `pass generate -c <path>`

![](cNxe2jsBpRftUKbujaB8LQ.webp)

You can even escape symbols, by passing a `-n` or `— no-symbols`flag

# Retrieving the password

Ok, now that you have stored the passwords, let’s finally learn to use them.

`pass -c <path>` will copy it to your clipboard for 45 secs. You’ll be prompted for your Passphrase and then, if you know, you know. 😏  
If you want cleartext, just remove the `-c`

![](IStUhXo86hHNF9i.webp)

# Listing the passwords

Do you need to remember that you asked Pass to remember the password of your favorite site? Nah! just ask Pass to list them with `pass`

![](C3Pg01S0c_u6CkkL_06l4A.webp)

I’m sure you’ve found Pass to be a really useful tool by now.🎉🎉🎉

# Further reading

check the [official website](https://www.passwordstore.org/) to know about

- Initializing Pass with git
- Storing metadata for password
- Pass extensions