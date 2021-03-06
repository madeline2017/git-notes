# git-notes
A small web app to let anyone easily append notes into a GitHub repo without needing to use Git or GitHub.

**Working around GitHub permissions:**

- Option 1: Have each user fork, commit, and make a pull request for every update (Potential issues: might break if multiple users make a commit at the same time!)

- Option 2: Add all users as collaborators on the base repo first (either manually or automate it for organizations with https://developer.github.com/v3/repos/collaborators/), and then have each user commit directly for every update (Potential issues: might cause merge conflicts, and also might break if multiple users make a commit at the same time!)

- Option 3: Make all commits directly from the moderator's account, using the optional "author" or "contributor" parameters to give credit for commits to each user's contribution. (Won't show up in each user's profile though. See https://help.github.com/articles/viewing-contributions-on-your-profile/)

- Option 4: Just make all commits from moderator's account and give credit by appending people's names (or support anonymous users too?) within the content of the notes, without giving them credit within GitHub's system.

- Option 5: Look into possibility of using a [GitHub Integration ](https://developer.github.com/early-access/integrations/) for this?

**Ideas for later features/workflow:**
- Let admins easily create a new notes file for each topic/meeting within the app.
- Allow all users to append to notes and view changes *in real time* with Firebase or WebSocket via server?
   - Temporarily save changes on server and/or in localStorage?
   - Should we make the forks/commits/PRs/merges upon each user contribution, or just make *one* commit at the end via the admin's GitHub account?

## Front-end stuff:
- Bare minimum: static web page with a log in with GitHub button, a text box to type in some notes, and button to save it
- Ideally: a nice Markdown editor/previewer, and more features to come!

## Back-end stuff:

### 1. Authenticate the user with GiHub

   Using https://github.com/prose/gatekeeper hosted on Heroku!

### 2. Fork the base repo containing the shared notes

   See API docs: https://developer.github.com/v3/repos/forks/#create-a-fork.
   
   Here's an example of testing this in cURL via command line:
   
   ```
   curl -i -H 'Authorization: token TOKEN-GOES-HERE' https://api.github.com/repos/LearnTeachCode/git-notes/forks -d ''
   ```
   
### 3. Get contents of an existing file

   See API docs: https://developer.github.com/v3/repos/contents/#get-contents
   
   Here's an example of requesting the raw contents of this file, testing this in cURL via command line:
   
   ```
   curl -i -H 'Authorization: token TOKEN-GOES-HERE' -H https://api.github.com/repos/LearnTeachCode/git-notes/contents/README.md
   ```
   
   Be sure to save the SHA of the file to use it in Step 4 below. **Note:** The file contents are encoded in Base64, so we can use the [`window.atob()` method](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/atob) in decode it (only available in IE 10+).

### 4. Commit to the repo to append user input to shared notes

   See API docs: https://developer.github.com/v3/repos/contents/#update-a-file
   
   After appending the user's input to the existing file contents and then encoding the final string in Base64 with the [`window.btoa()` method](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/btoa), here's an example of updating a file: 
   
   ```
   curl -i -X PUT -H 'Authorization: token TOKEN-GOES-HERE' -H https://api.github.com/repos/LearnTeachCode/git-notes/contents/test.md -d '{"path": "test.md", "message": "Test updating file via GitHub API", "content": "IyBUaGlzIGlzIGEgdGVzdCEKCkFuZCBoZXJlIGlzIHNvbWUgdXNlciBpbnB1dCBhcHBlbmRlZCB0byB0aGUgcHJldmlvdXMgZmlsZSBjb250ZW50cy4K", "sha": "SHA-FROM-STEP-3-GOES-HERE"}'
   ```
   
   Be sure to save the SHA from GitHub's response after updating the file, because the new SHA is required to make any future updates to the file.
   
   **For reference:** here's an outline of how to make a Git commit *the hard way* with the low-level Git Data API: https://github.com/LearnTeachCode/git-notes/blob/master/git-commit-test-steps.md

### 5. Create the pull request

   See API docs: https://developer.github.com/v3/pulls/#create-a-pull-request.
   
   Here's an example of testing this in cURL via command line:
   
   ```
   curl -i -H 'Authorization: token TOKEN-GOES-HERE' https://api.github.com/repos/LearnTeachCode/git-notes/pulls -d '{"title": "Test PR!", "body": "test", "base": "master", "head": "LearningNerd:master"}'
   ```

### 6. Display success message with the link to the newly-created pull request!
