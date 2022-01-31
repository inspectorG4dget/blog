# Slash Commands

Slash commands are a good way to get GitHub to respond to comments on a PR or issue. This is achieved through GitHub Actions (GHAs), which is part of GitHub's CI/CD pipeline. Following is an example of running comment-invoked CI/CD actions on the latest commit of a PR. GitHub automatically knows to look for these action definitions in `.github/workflows` of any repository. [Here's the authoritative GHA documentation][GHA documentation].

## GHA Definition

```
on:
  issue_comment:
    types: [created, edited]
name: Invoke Linter
jobs:
  check_comments:
    if: github.event.issue.pull_request && startsWith(github.event.comment.body, '/lint')  # this is a PR comment with /lint
    name: Check comments for /lint
    runs-on: ubuntu-latest
    steps:
      - name: Add rocket  # indicate that comment has been read and the workflow has been dispatched
        uses: actions/github-script@v3
        with:
          github-token: ${{secrets.GITHUB_TOKEN}}
          script: |
            github.reactions.createForIssueComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              comment_id: context.payload.comment.id,
              content: "rocket"
            })

      - name: Check out Git repository  # check out our code
        uses: actions/checkout@v2
        with:
          ref: refs/pull/${{ github.event.issue.number }}/head  # head of the PR

      - id: ref
        run: echo "::set-output name=sha::$(git rev-parse HEAD)"  # SHA of the most recent commit on this PR

      - name: Set pending status
        uses: geekodour/gh-actions-custom-status@v0.0.4
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          LAST_COMMIT_SHA: ${{ steps.ref.outputs.sha }}
        with:
          args: >-
            --state=pending
            --context=linting
            --target_url=https://github.com/path/to/repo/actions/runs/${{ github.run_id }}

      - name: Set up Python
        uses: actions/setup-python@v1
        with:
          python-version: 3.3

      - name: Install Python dependencies
        run: pip install flake8 flake8-print

      - name: Linting
        run: python -m flake8 --max-line-length 100

      - if: always()  # the linting step may have failed with errors
        name: Set linter output status
        uses: geekodour/gh-actions-custom-status@v0.0.4
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          LAST_COMMIT_SHA: ${{ steps.ref.outputs.sha }}
        with:
          args: >-
            --state=${{ job.status }}
            --context=linting
            --target_url=https://github.com/path/to/repo/actions/runs/${{ github.run_id }}
```

## Understanding the GHA definition

Let's take a closer look into the components of GHA definition.

1. `on:` defines when this action should run. In this case, it says that the action should run when there is an issue comment. **Point to note, here**: GitHub considers PRs to be issues in so far as GHA is concerned. So it doesn't matter that it says *issue*, it does also cover PRs.  
[Here's the documentation for the GHA triggers][GHA triggers]
2. In this case, we're triggering this workflow any time a new comment is created or an existing comment is edited. This is captured by `[created, edited]`. Each activity trigger has a list of activity types. The list of activity types for the `issue_comment` trigger can be found [here][issue_comment activity types].
3. `name` defines a monicker to be associated with this GHA. This is what is displayed when we look at the history of actions associated with a repository
4. `jobs` lists the specific workflows that need to be run when this GHA is triggered. Jobs run in parallel by default, but can be made to run sequentially by using dependencies. Each job also runs on a separate GHA runner (a virtual machine provided by GitHub), which means that running multiple jobs sequentially on the same runner requires them to be captured within a single job, under multiple steps (read on).
5. `runs-on` defines the platform (OS) on which this action should be run
6. `steps` is a list of operations to be performed as part of this job. These are run sequentially. Each step has a name (which is reported in the GHA progress/logs) and some directives of what to do.  
Here's where things get particularly interesting: the directive could be an in-line shell script, or it could be some pre-packaged directive made by someone else. GitHub has a [marketplace][GHA marketplace] for such pre-packaged actions.
   1. Pre-packaged actions can be invoked with `uses:`. If this action requires some parameters (usually mentioned in the documentation), those can be supplied in the `args:` block within a `with:` block. The `>-` is YAML syntactic sugar to [fold newlines into spaces][YAML multiline].  
   Sometimes, the GHA will require a parameter, and at other times, some environment variable needs to be set. Setting an environment variable is accomplished by listing them within an `env:` block.
   2. Running in-line shell scripts is quite simple, and can be accomplished with a `run:` block.
7. Branching is achieved with the `if:` block. This allows us to look for certain conditions within the newly-created or edited comment which has triggered this GHA. Remember how I previously said that GitHub treats issues and PRs as _issues_? Ok this is where we filter for ONLY PRs so we don't accidentally run this on an errant comment in an issue. This is done with `if: github.event.issue.pull_request`, and we can look for command sustrings with `startsWith(github.event.comment.body, '/lint')` and join the two together with `&&`.  
  There is a special `if: always()`, which is like saying `if True:` (in python). Typically, the final, cleanup steps go in this block.
8. Since the GHA runners aren't too powerful, we want some indication on the issue whether the command has been noticed, and whether the action is being run. 
   1. I choose to use the rocket reaction icon on the comment itself, to show that the comment has been noticed, and GitHub is doing things with it. This is achieved by the step named `Add rocket`.
   2. I also choose to use GitHub's native status indicator to display whether an action is pending or done (with success/failure). This is done in the steps named `Set pending status` and `Set linter output status`.
9. There's something to be said about variables in GHAs. Variables are referenced using the `${{ my_var_name }}` syntax. There are three types of variables:
   1. Native variables that don't need to be defined.  
   These are part fo the GHA runtime. These include `github.event.issue`, `github.event.comment`, `steps.my_step_name` (eg: `steps.ref`), `job.property` (eg: `job.status`), etc.
   2. Configured variables that need to be defined using GitHub's interface.  
   These include `secrets.GITHUB_TOKEN`, which must be configured separately using [GitHub's Secrets feature][GH secrets]
   3. Job/Step output variables that need to be captured and passed specifically.  
   This is specifically important, in order to use the output of a specific step. Since multiple steps may each have their own outputs, it is important to name a step with an ID. This is accomplished with the `id:` declaration within a `step:` block. The specific output for a step can be set by using a commandline `echo` with a `::set-output`, for example:  
   ```echo '::set-output name=sha::$(git rev-parse HEAD)"```  
   This sets the output of the given step in a variable named `sha` (multiple outputs can be set with different names), whose value is the output of `$(git rev-parse HEAD)` on the command line (since this is within a `run:` block).
     

[GHA triggers]:https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows
[issue_comment activity types]: https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#issue_comment
[GHA documentation]: https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobs
[GHA marketplace]: https://github.com/marketplace?type=actions
[YAML multiline]: https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html
[GH secrets]: https://docs.github.com/en/actions/security-guides/encrypted-secrets