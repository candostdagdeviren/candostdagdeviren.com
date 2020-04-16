---
layout: single
title: "Using SwiftLint and Danger for Swift Best Practices"
date: 2017-04-30
last_modified_at: 2020-04-16
categories: [Tools, Swift, Software Development, English]
tags: [Apple, automate, danger, Development, how to, lint, Software, Software Development, swift, swift-3, swift3, swiftlint]
header:
  overlay_filter: rgba(0, 0, 0, 0.44)
  overlay_image: assets/images/using-swiftlint-and-danger-for-swift-best-practices/cover.png
excerpt: "In a lot of ways, Swift is very flexible language and it is really easy to misuse it. Applying best practices becomes really important."
meta: "Apple's Swift Swift is becoming more and more popular amongst the developer community. Most of us already started adapting our projects to this folk."
read_time: true
share: true
related: true
---

Apple’s Swift is becoming more and more popular amongst the developer community. Most of us already started adapting our projects to this folk. While adopting, we may not be as careful as we should be as Swift is a very flexible language and it’s really easy to misuse it. Especially coming from an Objective-C culture, applying best practices becomes really important.

After reading [Swifty Tips from Göksel](https://medium.com/nsistanbul/swifty-tips-%EF%B8%8F-8564553ba3ec), I realized a couple of his tips can be checked automatically with [**SwiftLint**](http://bit.ly/2pszEcm). Also, we’re lazy people and we tend to forget to check our code before merging to master. Just in here, **[Danger](http://bit.ly/2pJ2DLY)** comes to the stage with shiny clothes and points out we’re doing something dangerous.

{:refdef: style="text-align: center;"}
![using-swiftlint-and-danger-for-swift-best-practices-1]({{ site.baseurl }}/assets/images/using-swiftlint-and-danger-for-swift-best-practices/1.gif)
{: refdef}

Sounds interesting..🤔 But what are these tools, actually?

# SwiftLint 💪🏽

[SwiftLint](http://bit.ly/2pszEcm) is an open source tool to enforce Swift style and conventions. It is developed by Realm. You can set your coding style rules and force them during development. SwiftLint has a command line tool, Xcode plugin, AppCode and Atom integration. So, it always fits your development environment. It’ll show you warnings and/or errors if you violate the linting rules.

{:refdef: style="text-align: center;"}
![using-swiftlint-and-danger-for-swift-best-practices-2]({{ site.baseurl }}/assets/images/using-swiftlint-and-danger-for-swift-best-practices/2.png)
{: refdef}

You can take a look at setup guide and tutorial from [here](http://bit.ly/2oJGIUS). After installation, you’ll have some rules by default. For example, it warns when you use private IBOutlet or force unwrapping in optionals.

Let’s take a look Göksel’s [tips](http://bit.ly/2oTbJkg). He says, "Never use implicitly unwrap optionals". SwiftLint provides this by default exactly how he describes. SwiftLint will warn you when you implicitly unwrap an optional except if it’s IBOutlet. The other one is "Avoiding _ misuse". SwiftLint is smart enough to point you out when you’re not using your bound optionals.

```swift
if let _ = Foo.optionalValue { } // Trigers a warning
if case .some(let _) = self {} // Triggers a warning
if foo() { let _ = bar() } // Not triggers a warning
if foo() { _ = bar() } // Not triggers a warning
```

In addition to applying best practices individually, we want to make the codebase consistent. Make it easier to apply custom rules. These rules should fit best practices, though. Configuring linting is handled from the *.swiftlint.yml* file. This file sits in project’s main path. We can enable, disable or write custom rules in this YML file. Let’s take a look at some examples.

First things first, writing a big function is generally a bad sign. If your function is getting bigger, it means that you should split [the responsibility](http://bit.ly/2qqFzxT). Add following code piece to your *.swiftlint.yml* file. This warns developers to have functions less than 200 lines. If programmer reaches 300, SwiftLint generates an error. Remember, you can ignore warnings but not errors. 😉


```ruby
function_body_length:
 - 200 # warning
 - 300 # error
```

Almost every project has dependencies or code pieces that are not possible to change. These code pieces should not be linted at all. For example, if a project uses CocoaPods as dependency manager, it’ll have Pods folder which keeps all dependency files. We need to exclude this folder from the linting process. As you can see below, it’s so easy.

```ruby
excluded:
  - Pods
```

{:refdef: style="text-align: center;"}
![using-swiftlint-and-danger-for-swift-best-practices-3]({{ site.baseurl }}/assets/images/using-swiftlint-and-danger-for-swift-best-practices/3.gif)
{: refdef}

As you saw from examples, what gives the extra boost to SwiftLint is flexibility. Sometimes you have to break the rules in special lines or files. These situations handled in SwiftLint with special comments. You can use the followings to adjust rules in these cases.

Add this comment to disable the rule in the file:

```swift
//swiftlint:disable rule_name
```

Add this comment to disable the rule in the following line:


```swift
//swiftlint:disable:next rule_name
```

Add this comment to disable the rule in the previous line:

```swift
//swiftlint:disable:previous rule_name
```

You can get the list of all rules by running `swiftlint rules` command in terminal.😏

Finally, we finalized our rules and now we can code in peace. But even some cases, you have to be more careful than just applying your linting rules. This is where Danger comes into place.

**P.S.:** You can find my predefined *.swiftlint.yml* file in [here](http://bit.ly/2pswdlX)😉.

# Danger ⚡️

Every project/piece of code has its own specific flow. When the project grows, maintaining and adding new features become harder. Error prone increases. Having coding guidelines and applying best practices are generally not enough. We are human, we make mistakes. [Danger](http://bit.ly/2pJ2DLY) can catch basic errors and let us think the harder problems. For example, it can catch common typos or generated file changes that you shouldn’t change by yourself. It can also force you to write tests if you write more than 20 lines of code. The rules are in your hands as same as SwiftLint.

Danger is a Ruby gem which runs in CI during pull request/merge request process. It leaves messages, comments or even fails your CI build when your rules are violated. Danger can run on several CI tools and can chat on GitHub, Bitbucket, and GitLab.

{:refdef: style="text-align: center;"}
![using-swiftlint-and-danger-for-swift-best-practices-4]({{ site.baseurl }}/assets/images/using-swiftlint-and-danger-for-swift-best-practices/4.png)
{: refdef}

For [single responsibility](http://bit.ly/2qqFzxT) and easier code review, developers shouldn’t open big pull requests. If a pull request has more than 600 lines of code, there should be a warning to split the pull request. Danger can provide this with a single line of configuration:

```ruby
warn Big PR, consider splitting into smaller if git.lines_of_code > 600
```

What else? If you’re working with the Test-After development process, you can easily forget to write tests. On the other hand, there should be automated way for "You forgot to add tests" comments. In general, if you change more than 20 lines of code, you should write tests. The number of lines depends on your decision, but you got the idea. Let’s take a look how we can achieve this with Danger:

```ruby
## Let's check if there are any changes in the project folder
has_app_changes = !git.modified_files.grep(/ProjectName/).empty?

## Then, we should check if tests are updated
has_test_changes = !git.modified_files.grep(/ProjectNameTests/).empty?

## Finally, let's combine them and put extra condition 
## for changed number of lines of code
if has_app_changes && !has_test_changes && git.lines_of_code > 20
  fail("Tests were not updated", sticky: false)
end
```

Danger is suitable for every kind of project. It provides a broad range of configurations to several languages by plugins. In Swift case, Ash Furrow developed [a plugin](http://bit.ly/2pvYyd2) for SwiftLint. Thanks to this plugin, we can have SwiftLint warnings as inline comments in the pull request. You can see installation guide [here](http://bit.ly/2pvYyd2). After installation, you’ll need to add following lines to end of your *Dangerfile*.

```ruby
swiftlint.lint_files
swiftlint.lint_files inline_mode: true
```

*Dangerfile* ensures your development guidelines applied to your code. It makes you more confident. In the long run, warnings teach you to be more careful. There is a reference guide in [here](http://bit.ly/2oCQ9pd) to give you more detailed view of Danger's capabilities.

**Note:** You don't have to configure CI. It's possible to run Danger on your local machine with `danger local` command.

Thanks to Eren’s [response](https://medium.com/@erenkabak/incredibly-good-post-especially-combination-with-danger-was-something-i-did-not-know-bb3932fa3449), if `danger local` command doesn’t run across the last open PR, you can always use following command:

```ruby
danger pr https://YOUR_PR_URL --dangerfile=YOUR_DANGERFILE_PATH
```

**P.S.:** You can find my predefined *Dangerfile* in [here](http://bit.ly/2oT8Z6s)😉.

# **Bonus:** SwiftLint with Git Hook

If you’re working with different text editors or IDEs which SwiftLint doesn’t support, you can only use command line tools to lint your code. This is an extra step and it’s easy to forget. Good thing, we can automate this. Hook feature in Git is another place to automate things. Basically, Git hooks are scripts where Git executes before or after events such as commit, push, and receive. We can run SwiftLint in one of these scripts. Personally, I’m using SwiftLint in the pre-commit hook while I’m writing Swift in Sublime Text.

**P.S.:** You can find my full pre-commit hook in [here](http://bit.ly/2oWmiE8)😉. If you want to use the same, just place the file above under .git/hooks folder inside your project. (You'll see sample hook scripts in there. Place it among them.) You can also use as a different hook. You can take a look at the list of available hooks and more information in [here](http://do.co/2qfn4x3).

## The End 😌

Let Danger and SwiftLint handle the trivial stuff for you. From now on, you can skip basic problems and focus on more complicated things during code review. SwiftLint and Danger ensure that everything is in place as you want. Wanna try?

{:refdef: style="text-align: center;"}
![using-swiftlint-and-danger-for-swift-best-practices-5]({{ site.baseurl }}/assets/images/using-swiftlint-and-danger-for-swift-best-practices/5.gif)
{: refdef}
