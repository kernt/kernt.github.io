We’ve been working **with gitlab runners for some time now**, we have a deploy server that we use to **register projects** and to make life easier for us, DevOps, on the **security point of view** (firewall configuration is easier when you use your own server instead of the whole public IP range from google cloud used on shared runners), also **we can scale up or down**, and we don’t **waste pipeline minutes on the free tier** (that is a fair reason if you make intensive use of CI/CD).

Since we have a lot of projects with CI/CD on these runners, I was tired on adding the runners manually, so I thought on some playbooks to automate the process

**If you are not familiar with custom runners, you can check the documentation** [**here**](https://docs.gitlab.com/runner/)

PD: I know that there are some Ansible modules dedicated to gitlab, but it didn’t address what I needed to.

# Requirements

You should have:

- **A personal access token with read access on repository and api (**[**How to generate it**](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html)**)**
- **Ansible ≥ 2.10**
- **A runner previously installed on docker-compose (easier to manage that bare server install)**
- **Some knowledge of Ansible**

# Let’s get to work!

## **_File structure_**

I have my files organized like this:

![](EGv0JOmE8ir2GmPy6YDXBg.webp)

I have an inventory which basically consist of localhost & the runner:

![](ad-N2ugsWxfUROKPZwW3uQ.webp)

I also have a variable file that applies to all host (not a good practice but since I do not have a complicated inventory is fine):

```yaml
api_token: your-token # api token to authenticate against gitlab
token_var_prefix: runner_token_ # prefix variable used on set_fact task to identify facts set with loop control

gitlab_external_url: https://gitlab.com # gitlab URL (change it if you use self-hosted)

gitlab_runner_docker_path: /docker/gitlab.docker.runner # path where your compose runner is

projects: # dictionary used to register the runner on the projects
  - id: xxxx # project ID
    tags: # Tags used on .gitlab-ci.yml to use the runner
      - deploy 
    description: "symfony-runner-1" # Runner name/description, must be unique
  - id: xxxxx
    tags:
      - deploy
    description: "flutter-runner-1"
```

Then I have **3 playbooks**, **main.yml will call the other two playbooks**, and it will loop over the “projects” dictionary

**main.yml**

```yaml
- hosts: localhost
  tasks:
  - name: Runner token
    include_tasks: utils/runner_token.yml
    loop: "{{ projects }}"
    loop_control:
      index_var: runner_loop_id

- hosts: runner
  tasks:
  - name: Register runners
    include_tasks: utils/runner_register.yml
    loop: "{{ projects }}"
    loop_control:
      index_var: runner_loop_id
      
  - name: Restart runner
    shell: docker-compose restart
    args:
      chdir: "{{ gitlab_runner_docker_path }}"
```

**runner_register.yml**

```yaml
---
  - name: Check if runner is registered already
    shell: |
        docker-compose exec gitlab-runner gitlab-runner list
    args:
      chdir: "{{ gitlab_runner_docker_path }}"
    register: check_runner   
  - name: Register runners
    when: item.description not in check_runner.stdout
    shell: |
      docker-compose exec gitlab-runner gitlab-runner register \
      --non-interactive \
      --url "{{ gitlab_external_url }}" \
      --registration-token "{{hostvars['localhost'][token_var_prefix+runner_loop_id|string]}}" \
      --description "{{ item.description }}" \
      --executor "docker" \
      --docker-privileged \
      --tag-list '{{ item.tags | join(",") }}' \
      --docker-image='{{ item.image |default('alpine') }}'
    args:
      chdir: "{{ gitlab_runner_docker_path }}"
```

**runner_token.yml**

```yaml
---
  - name: Get project details
    uri:
      url: "{{ gitlab_external_url }}/api/v4/projects/{{ item.id }}"
      headers:
        PRIVATE-TOKEN: "{{ api_token }}"
    register: project_output
  - name: Set runner token as fact
    set_fact:
      "{{ token_var_prefix+runner_loop_id|string }}": "{{project_output.json.runners_token}}"
```

## Okay, but what does what

Probably, you are asking yourself how the _f***_ I use these playbooks, well, I’ll do my best to explain it to you:

So, on the first call of **main.yml** I will iterate over the `projects` dictionary, and I will retrieve **the registration token for the runners**, then I will **set it as a fact** with this naming scheme: `"{{token_var_prefix+runner_loop_id|string }}": "{{ project_output.json.runners_token }}"` which will end up in something like: `runner_token_0` (note that the “0” comes from the `runner_loop_id` variable, which is set on main.yml with `loop_control`)

Then, on the second call **we use the same workflow and iterate over the same dictionary**, but this time we **connect to the runner and run the register command** (the one I use fits my needs, but make sure it fits yours!), we **use the same naming scheme in order to get the facts we set before**. Right before registering **we check the output of the list command and if we find a runner matching the description we do not register it** (avoid duplicate registers), and on the last task **we restart the runner to make sure changes are loaded.**

Finally, how you would execute it? (without hardcoding your api token)

`ansible-plabook -i inventory/deploys main.yml -e api_token=xxxxxx`

PD: Aditionally to make ssh faster create an ansible.cfg and add it to the root folder where you execute main.yml

## That’s all folks!

With this set of playbooks you will be able to register the runners automatically on any gitlab project you want, and keep track of the projects registered on those runners.

There are some updates to be done anyways to this set of playbooks, I would like to make a role which will allow:

- **register / unregister runners**
- **select which runners you want to register on which project**
- **Verify the runners are configured correctly**
- **Choose which options to add on the register command**

But for my needs it’s good for now.