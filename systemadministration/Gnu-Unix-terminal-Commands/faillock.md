Unlocking Account Using faillock
To unlock a user, we can call faillock with the –reset flag. Combining this with the –user flag unlocks a specific user.

Let’s use that on the user baeldung:

`faillock --user baeldung  --reset`

Copy
This command doesn’t return any output when it succeeds.

Unlocking Account Using /var/run/faillock File
Sometimes there can be a situation where it’s easiest to alter the filesystem to unlock a user. If so, we can delete the files that faillock uses to track a user’s login attempts.

Let’s look at those files as they existed in the example above. The default directory in which faillock stores these files is /var/run/faillock. Listing them with ls shows:

`ls /var/run/faillock`

baeldung
user
Copy
This shows logs for the user and baeldung.

