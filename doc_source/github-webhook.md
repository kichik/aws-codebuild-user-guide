# GitHub webhook events<a name="github-webhook"></a>

You can use webhook filter groups to specify which GitHub webhook events trigger a build\. For example, you can specify that a build is only triggered for changes to specific branches\. 

You can create one or more webhook filter groups to specify which webhook events trigger a build\. A build is triggered if all the filters on one or more filter groups evaluate to true\. When you create a filter group, you specify: 

**An event**  
For GitHub, you can choose one or more of the following events: `PUSH`, `PULL_REQUEST_CREATED`, `PULL_REQUEST_UPDATED`, `PULL_REQUEST_REOPENED`, and `PULL_REQUEST_MERGED`\. The webhook event type is in the `X-GitHub-Event` header in the webhook payload\. In the `X-GitHub-Event` header, you might see `pull_request` or `push`\. For a pull request event, the type is in the `action` field of the webhook event payload\. The following table shows how `X-GitHub-Event` header values and webhook pull request payload `action` field values map to the available event types\.      
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/codebuild/latest/userguide/github-webhook.html)
 The `PULL_REQUEST_REOPENED` event type can be used with GitHub and GitHub Enterprise Server only\. 

**One or more optional filters**  
Use a regular expression to specify a filter\. For an event to trigger a build, every filter associated with it must evaluate to true\.     
`ACTOR_ACCOUNT_ID` \(`ACTOR_ID` in the console\)  
A webhook event triggers a build when a GitHub or GitHub Enterprise Server account ID matches the regular expression pattern\. This value is found in the `id` property of the `sender` object in the webhook payload\.  
`HEAD_REF`  
A webhook event triggers a build when the head reference matches the regular expression pattern \(for example, `refs/heads/branch-name` or `refs/tags/tag-name`\)\. For a push event, the reference name is found in the `ref` property in the webhook payload\. For pull requests events, the branch name is found in the `ref` property of the `head` object in the webhook payload\.   
`BASE_REF`  
A webhook event triggers a build when the base reference matches the regular expression pattern \(for example, `refs/heads/branch-name`\)\. A `BASE_REF` filter can be used with pull request events only\. The branch name is found in the `ref` property of the `base` object in the webhook payload\.  
`FILE_PATH`  
A webhook triggers a build when the path of a changed file matches the regular expressions pattern\. A `FILE_PATH` filter can be used with GitHub push and pull request events and GitHub Enterprise Server push events\. It cannot be used with GitHub Enterprise Server pull request events\.   
`COMMIT_MESSAGE`  
A webhook triggers a build when the head commit message matches the regular expression pattern\. A `COMMIT_MESSAGE` filter can be used with GitHub push and pull request events and GitHub Enterprise Server push events\. It cannot be used with GitHub Enterprise Server pull request events\.

**Note**  
You can find the webhook payload in the webhook settings of your GitHub repository\. 

**Topics**
+ [Filter GitHub webhook events \(console\)](#github-webhook-events-console)
+ [Filter GitHub webhook events \(SDK\)](#github-webhook-events-sdk)
+ [Filter GitHub webhook events \(AWS CloudFormation\)](#github-webhook-events-cfn)

## Filter GitHub webhook events \(console\)<a name="github-webhook-events-console"></a>



In **Primary source webhook events**, select the following\. This section is only available when you chose **Repository in my GitHub account** for the source repository\.

1. Select **Rebuild every time a code change is pushed to this repository** when you create your project\. 

1. From **Event type**, choose one or more events\. 

1. To filter when an event triggers a build, under **Start a build under these conditions**, add one or more optional filters\. 

1. To filter when an event is not triggered, under **Don't start a build under these conditions**, add one or more optional filters\. 

1. Choose **Add filter group** to add another filter group, if needed\. 

 For more information, see [Create a build project \(console\)](create-project-console.md) and [WebhookFilter](https://docs.aws.amazon.com/codebuild/latest/APIReference/API_WebhookFilter.html) in the *AWS CodeBuild API Reference*\. 

In this example, a webhook filter group triggers a build for pull requests only:

![\[\]](http://docs.aws.amazon.com/codebuild/latest/userguide/images/pull-request-webhook-filter.png)

Using an example of two webhook filter groups, a build is triggered when one or both evaluate to true:
+ The first filter group specifies pull requests that are created, updated, or reopened on branches with Git reference names that match the regular expression `^refs/heads/main$` and head references that match `^refs/heads/branch1$`\. 
+ The second filter group specifies push requests on branches with Git reference names that match the regular expression `^refs/heads/branch1$`\. 

![\[\]](http://docs.aws.amazon.com/codebuild/latest/userguide/images/pull-request-webhook-filter-head-base-regexes.png)

In this example, a webhook filter group triggers a build for all requests except tag events\. 

![\[\]](http://docs.aws.amazon.com/codebuild/latest/userguide/images/pull-request-webhook-filter-exclude.png)

In this example, a webhook filter group triggers a build only when files with names that match the regular expression `^buildspec.*` change\. 

![\[\]](http://docs.aws.amazon.com/codebuild/latest/userguide/images/pull-request-webhook-filter-file-name-regex.png)

In this example, a webhook filter group triggers a build only when a change is made by a specified GitHub or GitHub Enterprise Server user with an account ID that matches the regular expression `actor-account-id`\. 

**Note**  
 For information about how to find your GitHub account ID, see https://api\.github\.com/users/*user\-name*, where *user\-name* is your GitHub user name\. 

![\[\]](http://docs.aws.amazon.com/codebuild/latest/userguide/images/pull-request-webhook-filter-actor.png)

In this example, a webhook filter group triggers a build for a push event when the head commit message matches the regular expression `\[CodeBuild\]`\. 

![\[\]](http://docs.aws.amazon.com/codebuild/latest/userguide/images/pull-request-webhook-filter-commit-message.png)

## Filter GitHub webhook events \(SDK\)<a name="github-webhook-events-sdk"></a>

To use the AWS CodeBuild SDK to filter webhook events, use the `filterGroups` field in the request syntax of the `CreateWebhook` or `UpdateWebhook` API methods\. For more information, see [WebhookFilter](https://docs.aws.amazon.com/codebuild/latest/APIReference/API_WebhookFilter.html) in the *CodeBuild API Reference*\. 

 To create a webhook filter that triggers a build for pull requests only, insert the following into the request syntax: 

```
"filterGroups": [
   [
        {
            "type": "EVENT", 
            "pattern": "PULL_REQUEST_CREATED, PULL_REQUEST_UPDATED, PULL_REQUEST_REOPENED, PULL_REQUEST_MERGED"
        }
    ]
]
```

 To create a webhook filter that triggers a build for specified branches only, use the `pattern` parameter to specify a regular expression to filter branch names\. Using an example of two filter groups, a build is triggered when one or both evaluate to true:
+ The first filter group specifies pull requests that are created, updated, or reopened on branches with Git reference names that match the regular expression `^refs/heads/main$` and head references that match `^refs/heads/myBranch$`\. 
+ The second filter group specifies push requests on branches with Git reference names that match the regular expression `^refs/heads/myBranch$`\. 

```
"filterGroups": [
    [
        {
            "type": "EVENT", 
            "pattern": "PULL_REQUEST_CREATED, PULL_REQUEST_UPDATED, PULL_REQUEST_REOPENED"
        },
        {
            "type": "HEAD_REF", 
            "pattern": "^refs/heads/myBranch$"
        },
        {
            "type": "BASE_REF", 
            "pattern": "^refs/heads/main$"
        }
    ],
    [
        {
            "type": "EVENT", 
            "pattern": "PUSH"
        },
        {
            "type": "HEAD_REF", 
            "pattern": "^refs/heads/myBranch$"
        }
    ]
]
```

 You can use the `excludeMatchedPattern` parameter to specify which events do not trigger a build\. For example, in this example a build is triggered for all requests except tag events\. 

```
"filterGroups": [
    [
        {
            "type": "EVENT", 
            "pattern": "PUSH, PULL_REQUEST_CREATED, PULL_REQUEST_UPDATED, PULL_REQUEST_REOPENED, PULL_REQUEST_MERGED"
        },
        {
            "type": "HEAD_REF", 
            "pattern": "^refs/tags/.*", 
            "excludeMatchedPattern": true
        }
    ]
]
```

You can create a filter that triggers a build only when files with names that match the regular expression in the `pattern` argument change\. In this example, the filter group specifies that a build is triggered only when files with a name that matches the regular expression `^buildspec.*` change\. 

```
"filterGroups": [
    [
        {
            "type": "EVENT", 
            "pattern": "PUSH"
        },
        {
            "type": "FILE_PATH", 
            "pattern": "^buildspec.*"
        }
    ]
]
```

You can create a filter that triggers a build only when a change is made by a specified GitHub or GitHub Enterprise Server user with account ID `actor-account-id`\. 

**Note**  
 For information about how to find your GitHub account ID, see https://api\.github\.com/users/*user\-name*, where *user\-name* is your GitHub user name\. 

```
"filterGroups": [
    [
        {
            "type": "EVENT", 
            "pattern": "PUSH, PULL_REQUEST_CREATED, PULL_REQUEST_UPDATED, PULL_REQUEST_REOPENED, PULL_REQUEST_MERGED"
        },
        {
            "type": "ACTOR_ACCOUNT_ID", 
            "pattern": "actor-account-id"
        }
    ]
]
```

You can create a filter that triggers a build only when the head commit message matches the regular expression in the pattern argument\. In this example, the filter group specifies that a build is triggered only when the head commit message of the push event matches the regular expression `\[CodeBuild\]`\. 

```
"filterGroups": [
    [
        {
            "type": "EVENT",
            "pattern": "PUSH"
        },
        {
            "type": "COMMIT_MESSAGE",
            "pattern": "\[CodeBuild\]"
        }
    ]
]
```

## Filter GitHub webhook events \(AWS CloudFormation\)<a name="github-webhook-events-cfn"></a>

 To use an AWS CloudFormation template to filter webhook events, use the AWS CodeBuild project's `FilterGroups` property\. The following YAML\-formatted portion of an AWS CloudFormation template creates two filter groups\. Together, they trigger a build when one or both evaluate to true: 
+  The first filter group specifies pull requests are created or updated on branches with Git reference names that match the regular expression `^refs/heads/main$` by a GitHub user who does not have account ID `12345`\. 
+  The second filter group specifies push requests are created on files with names that match the regular expression `READ_ME` in branches with Git reference names that match the regular expression `^refs/heads/.*`\. 
+ The third filter group specifies a push request with a head commit message matching the regular expression `\[CodeBuild\]`\.

```
CodeBuildProject:
  Type: AWS::CodeBuild::Project
  Properties:
    Name: MyProject
    ServiceRole: service-role
    Artifacts:
      Type: NO_ARTIFACTS
    Environment:
      Type: LINUX_CONTAINER
      ComputeType: BUILD_GENERAL1_SMALL
      Image: aws/codebuild/standard:4.0
    Source:
      Type: GITHUB
      Location: source-location
    Triggers:
      Webhook: true
      FilterGroups:
        - - Type: EVENT
            Pattern: PULL_REQUEST_CREATED,PULL_REQUEST_UPDATED
          - Type: BASE_REF
            Pattern: ^refs/heads/main$
            ExcludeMatchedPattern: false
          - Type: ACTOR_ACCOUNT_ID
            Pattern: 12345
            ExcludeMatchedPattern: true
        - - Type: EVENT
            Pattern: PUSH
          - Type: HEAD_REF
            Pattern: ^refs/heads/.*
          - Type: FILE_PATH
            Pattern: READ_ME
            ExcludeMatchedPattern: true
        - - Type: EVENT
            Pattern: PUSH
          - Type: COMMIT_MESSAGE
            Pattern: \[CodeBuild\]
```