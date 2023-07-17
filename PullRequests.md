## How To Start Contributing through a Pull Request:

1. Fork the repository you need to contribute to and keep the same name, though it can be changed.

2. Clone this new fork using `git clone ...`.

4. Check out your branch as follows:
  `git checkout -b <your_desired_branch_name>`.

5. Check that you are working on your desired branch as follows:
   `git branch`.

6. Make the necessary changes.

7. `git status`: check modified/added files.
   `git add <path/file1> <path/file2> ...`: stage the files.
   `git commit -m "your message here."`: commit to your changes.

8. `git push` does not work yet because branch does not exist. If this is tried, it will suggest you to run a command:
   `git push --set-upstream origin <your_desired_branch_name>`. So, run that to define your branch and push the changes.

9. Go back to your repository on Github.com, and you should be able to create a pull request.

Credits go to this youTube video: https://www.youtube.com/watch?v=jRLGobWwA3Y
