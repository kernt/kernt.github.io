# Introduction

In the realm of IT automation, timing and scheduling tasks are crucial for managing systems efficiently. Ansible, a powerful automation tool, facilitates this through various functions, including the `now()` function, which is an essential tool for fetching the current time within playbooks. This function, introduced in Ansible 2.8, utilizes the Jinja2 templating engine to retrieve a Python `datetime` object or a string representation of the current time, providing flexibility and precision in timing operations. This article will explore the `now()` function in detail, highlighting its usage and practical applications within Ansible automation tasks.

# Overview of the `now()` Function

The `now()` function is a Jinja2 extension that Ansible playbooks can use to obtain the current time. This function is incredibly versatile, supporting arguments to adjust the time zone to UTC and to format the datetime string according to specific needs.

## Arguments Supported by `now()`

- `utc`: By setting this argument to `True`, you can obtain the current time in UTC. The default value is `False`, which returns the local time of the system where the playbook is executed.
- `fmt`: This argument accepts a `strftime` compatible string, allowing the user to format the returned datetime object into a string formatted according to the provided pattern.

# Practical Examples

Let’s delve into some practical examples of how the `now()` function can be used in Ansible playbooks.

## Example 1: Fetching Current Time in UTC

To get the current time in UTC and format it into a readable string, you can use the `now()` function as follows:

```yaml
- name: Get current time in UTC  
  ansible.builtin.debug:  
    msg: "Current time (UTC): {{ now(utc=true, fmt='%Y-%m-%d %H:%M:%S') }}"
```

This example demonstrates fetching the current UTC time and formatting it in the `YYYY-MM-DD HH:MM:SS` format, which is commonly used for logging and time-stamping tasks.

## Example 2: Displaying Host Uptime

Ansible’s `now()` function can also be used to calculate and display a host's uptime in a human-readable format, such as days, hours, minutes, and seconds. Assuming facts have been gathered beforehand, you can use the `now()` function in combination with the host's uptime information:

```yaml
- name: Show the uptime in days/hours/minutes/seconds  
  ansible.builtin.debug:  
    msg: "Uptime {{ now().replace(microsecond=0) - now().fromtimestamp(now(fmt='%s') | int - ansible_uptime_seconds) }}"
```

This playbook snippet calculates the uptime by subtracting the host’s uptime in seconds from the current time, resulting in a timedelta object that represents the host’s uptime duration.