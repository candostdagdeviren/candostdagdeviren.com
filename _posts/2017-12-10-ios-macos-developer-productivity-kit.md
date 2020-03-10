---
layout: single
title: "iOS/macOS Developer Productivity Kit"
date: 2017-12-10
categories: [Tools, Software Development, English]
tags: [Apple, Best Practices, danger, Development, iOS, MVVM, productivity, Software, Software Development, swift, swiftlint]
excerpt: "We use some tools and also some do customizations on them to increase our productivity. Every tech stack has different needs, thus, a different set of tools."
featured: false
hidden: false
---

We use some tools and also some do customizations on them to increase our productivity. Every tech stack has different needs, therefore, a different set of tools. iOS, macOS, watchOS, tvOS developers are stuck in Xcode. Hence, we need to improve our productivity by customizing Xcode and using some external tools. This post is mostly about Xcode and I’ll mention some other tools that I’ve been using for some time and happy with them.

Let’s start with Xcode.

# Xcode

## Default Xcode Shortcuts

- When the project grows, searching time for the file that we’re currently coding in *Project Navigator* increases. The day I learned the ⌘⇧J shortcut to jump to the current file in navigation, I had a smile on my face.
- Who doesn’t love to work with *Assistant Editor* in Xcode? Some of us feel disturbed when they reached to mouse while coding. So, moving the focus between Assistant and Main Editors as well as tabs has a shortcut ⌘J. It shows you a window that you can choose the focused editor by arrows. But there is one more shortcut which I find more useful is ⌃`. This allows you to move your focus directly to the next editor.
- Sometimes I found myself copying and pasting a block of code couple of lines up and down. Recently, I learned there are two shortcuts which most developers don’t use (at least I didn’t see anyone using lately). For moving the line up ⌥⌘[ and moving the line down ⌥⌘]
- Everyone knows ⌘⇧O to open file search bar in Xcode. If you’re working with Objective-C and Swift in the same project, you can search in Swift Generated Headers in this search. When search bar is open, press⌘G to enable Swift generated headers. It’ll generate headers in Swift for Objective-C files.
- Let’s take a look at these shortcuts for opening/closing parts in Xcode that I’ve learned years ago from [StackOverflow](https://stackoverflow.com/a/8310524/1205160).

{:refdef: style="text-align: center;"}
![iosmacosproductvitykit-xcode]({{ site.baseurl }}/assets/images/ios-macos-developer-productivity-kit/xcode.png)
{: refdef}

During debugging, when we stopped the running code with a breakpoint, we can use ⌘⌃Y to *Step Over* as well as ⌘Y to enable/disable all breakpoints.

## Customized Shortcuts

Let’s focus on creating our own shortcuts. Most of us even don’t know that we can customize shortcuts in Xcode. Refactoring code is easier than ever with Xcode 9. And I’m not only talking about new renaming feature. I'm talking about all Refactor menu.

### How?

First, open *Properties* -like most of the macOS apps- using ⌘,. Switch to *Key Bindings* section for navigation. Choose *All* submenu. Now, you can see all shortcuts in there.

### Customize

Below you can see all customized shortcuts that I’ve been using for some time. Some of them conflict with Xcode default shortcuts and I decided to remove default ones because I never use them. You’ll also see some empty ones. Those are the ones I’m not using, conflicting or pressing by mistake all the time like *Print* in the beginning.

{:refdef: style="text-align: center;"}
![iosmacosproductvitykit-refactor]({{ site.baseurl }}/assets/images/ios-macos-developer-productivity-kit/refactor.png)
{: refdef}

You probably noticed that I changed the *Rename* shortcut to use same key combinations with other refactor methods.

## Debugger

I won’t get into details of LLDB debugger commands. I just want to suggest [a good tutorial](https://www.objc.io/issues/19-debugging/lldb-debugging/) on how to use the commands and [one great cheatsheet for LLDB commands](https://www.nesono.com/sites/default/files/lldb%20cheat%20sheet.pdf). One thing I want to mention is that if you get the following error while trying to access frame or similar things

> error: property 'frame not found on object of type 'UIView *'
> error: 1 errors parsing the expression

you should write expr import UIKit in LLDB command line. One way to execute this automatically is to create an auto-continue breakpoint in AppDelegate and add this command into *Debugger Command* action.

# Other Tools and Frameworks

## Alfred

I’ve replaced *Spotlight Search* with Alfred on my mac and I’m really happy with all the features it has. Alfred has a **Clipboard** and **Snippets** features that I’m using all day long while coding. I’ve created some snippets according to my current iOS application development stack. For example, we’re using MVVM architecture and I have boilerplate code snippets for ViewModel, View, ViewController that I can easily use. You can reach my Alfred Snippets for Swift from [here](https://github.com/candostdagdeviren/Productivity/blob/master/Swift.alfredsnippets).

## Command Line Tools & Aliases

- I like using Git from the command line instead of programs like SourceTree. I’ve been using **[Tig](https://jonas.github.io/tig/)** for that and I’m really happy so far. It’s easy & fast. Learning the tool can take some time but after you get used to it, you’ll see how easy to use Git.
- Also, we’re using branching model from [Vincent Driessen](http://nvie.com/posts/a-successful-git-branching-model/) at work. Of course, it has a tool that makes this usage easier. It’s called **[gitflow](https://github.com/nvie/gitflow)**.
- Lastly, I’m using **[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh/)** with some plugins (git zsh-autosuggestions docker git-flow tig swiftpm sublime osx history dotenv). You can choose the plugins that suitable for your tech stack.

## Third Party Libraries

I just want to mention some libraries that can save your time, literally.

**[Marathon](https://github.com/JohnSundell/Marathon)**, **[TestDrive](https://github.com/JohnSundell/TestDrive)**, **[SwiftLint](https://github.com/realm/SwiftLint/), [Danger](https://github.com/danger/danger)**

I have another **[blog post](/using-swiftlint-and-danger-for-swift-best-practices/)** about using SwiftLint and Danger check it out.

## Last But Not Least - OS-wide shortcuts

⌘⌥⎋ - Force Quit

⌘` - Change window for same application

## Final words

Time is the most important thing in our lives. Increase your productivity. Work efficiently. Use one tip or all. Actually, customize those tips for yourself. If you know cool shortcuts or other tools that boost your productivity, share it with me in the comments!

----

### Symbols

- Option - ⌥
- Command - ⌘
- Shift - ⇧
- Esc - ⎋
- Ctrl - ⌃
- Enter - ⏎

----
Thanks for reading! Help spread the word.❤️🚀 Have questions, suggestions, comments? Write a comment! 😍
