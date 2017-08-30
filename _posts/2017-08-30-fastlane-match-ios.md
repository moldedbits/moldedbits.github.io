---
layout: post
title:  "Fastlane Match - Matchmaker for iOS certificates and profiles"
date:   2017-08-30 08:00:00
author: amit
categories: Technical iOS
---

iOS certificates and Provisioning profiles, most avoided topic for all iOS developer, specially for all the beginners out there. From iOS 10 onwards signing is required, even for development build and running your application on simulator and devices. So it becomes very important for us to know how to manage our Certificates and Provisioning profiles.

### Single Developer / Single Project
If you are a single developer or working alone on a project and have your own apple developer account which you use for your development, you have nothing to worry about. Xcode8 introduced `Automatic Manage Signing` option. With this option enabled all your certificates and profiles are managed by Xcode itself and it just work perfect.

### Team / Multiple Project
If you are working in a team on a project then these are the options you have for Certificates and Provisioning profile management :-
1. Each developer have access to your single developer account and enable `Automatic Signing`. This way a new certificate gets generated on developer account respective to developer machines. And corresponding provisioning profile is generated and used. The downside for this is, one have to create a separate `Distribution` certificate and profile and share with all developers (If you want any one of them to release the build)
2. A single `Development` and `Distribution` certificate generated and shared with all developers in your team. Provisioning profile for each of new project needs to be generated and shared separately. Downside, if any new device is added or certificate get's revoked you have to repeat the complete process again. And this is just not possible with increasing team sizes and multiple projects.

### Fastlane to rescue

As we all know, in last couple of year `Fastlane` toolchain took us all with surprise and made all developers life cool again. Today, we are going to talk about specific tool from `fastlane` toolset, `Match`.

### What is Match?

I am going to borrow the `Match` introduction from `fastlane` git repo.

"A new approach to iOS code signing: Share one code signing identity across your development team to simplify your codesigning setup and prevent code signing issues."

"`match` is the implementation of the [Codesiging Guide](https://codesigning.guide) concept. match creates all required certificates & provisioning profiles and stores them in a separate git repository. Every team member with access to the repo can use those credentials for code signing. match also automatically repairs broken and expired credentials. It's the easiest way to share signing credentials across teams."

Above introduction explains why we should use `match` and you can get in-depth details about how `match` works underneath and security consideration using `match` at [Codesiging Guide](https://codesigning.guide). With `match`, managing Certificate and Provisioning profile becomes as easy as eating an apple pie (remember from [last post]( {{ site.baseurl }}{% post_url 2017-08-28-typed-notification-ios %} )

One of the best part of using `match` for your certificate management is, for your CI servers you don't have to manage certificates and provisioning profiles. We can have `fastlane` take care this for us and just add `match` to our release or `beta` lane.

## Getting started with Match

We are going to discuss the most simple and straightforward `match` usage, which will be the case for most the teams. To start with `match`, we have to first destroy all our existing certificates and profiles, so that we can start with clean slate. There is another setup, where you can use your existing certificates and profiles. More about that can be found [here](http://macoscope.com/blog/simplify-your-life-with-fastlane-match/#migration).

Before getting started we have to :-
1. Create a private git empty repo, this will used to store match generated Certificates and Provisioning profiles.
2. Get your shared Apple developer account e-mail and password.

### Match Setup
1. If you have existing Certificates and profiles on this account, you should consider using `match nuke`

    To clean existing certificates and profiles(with caution):-
    `fasltane match nuke` // Only for first time, when setting up match

2. Go to your project root folder and run `fastlane match init`, this will create a  `Matchfile` in fastlane folder (assuming you are already using fastlane)
3. You will be asked for your git repo url, you created earlier and your `Matchfile` will have this content :-
    ```ruby
    git_url <URL_TO_YOUR_GIT_REPO_FOR_CERTIFICATES>

    app_identifier <BUNDLE_ID>

    username <APPLE_DEVELOPER_USERNAME> # Your Apple Developer Portal username
    ```
    You can also create the same manually.
4. Now run `fastlane match development`, and this will create Development certificate and provisioning profile for the BUNDLE_ID and push it to git repo.

    You will be asked for a `PASSPHRASE`, this is your `MATCH_PASSWORD`. This will be used to encrypt all files with `openssl` before storing to git repo.

    Same you can do for `adhoc, appstore, and enterprise`.
5. Above will also install all these certificates in your machine. Commit the `Matchfile` to your source control.
Now your fellow teammate can install all the certificates and profiles by running `fastlane match development --readonly` and enter the same PASSPHRASE, you created. This will install `Development` certificate and profile, same can be done for `adhoc, appstore, and enterprise`.

    We used `--readonly` to be on safe side, that your fellow developer don't update the certificate and profiles.

And that's it. You are done. No manual certificates sharing, no profile sharing and exporting the same. How cool it that? Supercool!

This is basic setup, you can find other options with `fastlane match --help`

### Adding new project's profile

After you have setup your `match` and create your certificates and profile for a project, you can create provisioning profile for other projects and add it to the same repo. Just follow the step from 2 to 5.

This time your `Matchfile` will have BUNDLE_ID for your new project, and everything else remains the same.

### Common issues
- Sometime calling `fastlane match developement` or for other type, we get an error similiar to `Provisioning profile 'xxxxxxx' is not available on the Developer Portal`. This problem generally occurs when you delete some profile manually from your developer portal. No need to panic, solution for this is very simple.

    Just remove the profiles from your git repo, which were deleted from developer portal and commit the same.

    Your certificates git repo structure will be something like this
    ```
      ├── README.md
      ├── certs
      │   ├── development
      │   │   ├── <TEAM_ID>.cer
      │   │   └── <TEAM_ID>.p12
      │   └── distribution
      │       ├── <TEAM_ID>.cer
      │       └── <TEAM_ID>.p12
      ├── match_version.txt
      └── profiles
          ├── appstore
          │   ├── AppStore_<BUNDLE_ID_1>.mobileprovision
          │   └── AppStore_<BUNDLE_ID_2>.mobileprovision
          └── development
              ├── Development_<BUNDLE_ID_1>.mobileprovision
              └── Development_<BUNDLE_ID_2>.mobileprovision  //Deleted profile
    ```
- If your team were using `Automatic Signing` option previously and now moving to `match`, after `match` setup and profile installation, you will see errors in your Xcode certificates and profile section. This happens because for the same account you have installed two certificates, one create by Xcode Auto signing for your machine and one by `match`. Just delete the certificate create by Xcode Auto signing. (Find out this by your developer portal)
- Multiple project setup, we can supply multiple BUNDLE_ID as string array in `Matchfile` or we can also do the same without `Matchfile` from command line.

### Inspiration

As an iOS lead, I was facing problem for certificates and provisioning profile management. For a new project or a new member joining your team, `match` makes this super easy to create and install certificates. We started trying out `match` last year and also setup the same with our CI server, and It's just working perfect for our team. Now's the time to share the knowledge, so that others can take benefit from our learnings.

Happy automating!

The moldedbits Team
