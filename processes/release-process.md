# Release Checklist

## General Procedure

### Release Schedule

1. Minors and Patches will be released biweekly
2. Release Candidates will be released after first .net release-candidate of an LTS version is available
3. Majors will be release after a new .net LTS verison is available

### Procedure for a Release

For a Major release

1. Release a Layer
1. Update layers building on it
1. Release updated layer
1. Publish release announcement if all layers are updated and released

This is because all layers reference CI builds of the future branches before a major release. For minor releases there is no "sequence". If you have a patch or minor change for a layer, make a release as described below and **don't** update the version in other repositories, if not necessary. In this way, all packages reference the lowest possible patch of a major, which makes handling of the packages easier.

## How to release

### If Minor Release

#### Prepare Branches (**Only** required for the current major version)

- [ ] Checkout dev and fetch + pull
- [ ] Check C# & NPM Dependencies  
(main should only depend on released packages. So if you have updated a dependency and tested with a CI package on dev, update that now)
  - Package.json for Angular projects
  - Directory.Builds.Targets for C# projects
  - Rebuild your projects once and Push your changes
  - [ ] If updates needed, create a *Update packages to release versions* commit to **dev** and push to origin
- [ ] Checkout main (fetch & pull)
- [ ] Merge origin/dev into main
  - Command: `git merge origin/dev --no-ff`
  - Use `--no-ff` to get definitely a merge commit
  - Merge from origin to avoid a merge of an outdated local dev
  - Merge commit should look like: *Merge remote-tracking branch 'origin/dev'*
- [ ] Push to main `git push`
- [ ] Create *Bump Version* commit (message should be `Bump version x.y.z`) by updating the corresponding file to `MAJOR.MINOR.PATCH` on **dev** (e.g.  `VERSION: 1.2.4` after `Tag/Release: 1.2.3` was released, note the missing v here) and commit + push
  - `Package.json` for Angular projects (MORYX-Web -> moryx-web/projects/ngx-web-framework/package.json)
  - `VERSION` file for C# projects
- [ ] Create Milestone for **new** Version `ProjectNameWithoutPrefix MAJOR.MINOR.PATCH` on GitHub or GitLab  
*Note: This version must be identical to the new VERSION-file content*

#### Create Release

**Github Repository**
Go to *Releases -> Draft a new Release*

![Create GitHub Release](../images/github-release.png)

- [ ] Verify that the Pipeline ran without issues and that a new package exists on the feeds
  - [release](https://www.nuget.org/packages/Moryx) feeds for C# projects
  - [moryx-npm release](https://www.myget.org/F/moryx/npm) feed for Angular projects
- [ ] Copy URL of the release in the repository to release Anouncements

#### Clean-Up (Not needed for Major release)

- [ ] Checkout future
- [ ] Fetch + Pull
- [ ] Merge origin/dev to future (Keep future VERSION file)  
  - `git merge origin/dev`
  - Getting a merge conflict here is a good sign

### If Major Release

- [ ] If necessary, make a Minor release first and return here when you are done
  - I.e., if you have unreleased changes on dev
- [ ] Verify that a *Migration Guideline* for the release exists under `/docs/migrations`
  - If necessary, create a file `vX_tovY.md`
  - Check if there were any breaking changes and list them in the file  
    *Use a tool of your choice and compare `origin/dev` and `origin/future` branch to view all changes*
  - Add information about any breaking changes that require an application developer to change something in their code  
    **Remember that another person should be able to update the packages in their project only using the Migration Guideline**
- [ ] Checkout dev and fetch + pull
- [ ] Create branch `release/MAJOR` (e.g. `release/3`) for the last Major and push it
- [ ] Checkout `future` (fetch & pull)
- [ ] Check dependencies (dev should later only depend on CI or released packages)
  - Package.json for Angular projects
  - Directory.Builds.Targets for C# projects
  - global.json for C# projects
  - Rebuild you projects once and Push your changes
- [ ] Checkout `dev`
- [ ] Merge origin/future into dev
  - `git merge origin/future`
- [ ] Push dev
- [ ] Follow [minor release instructions](#if-minor-release) and return here when you are done
- [ ] Create *Bump Version* commit (message should be `Bump version x.y.z`) by updating the corresponding file to `MAJOR.0.0` on **future** (e.g.  VERSION: 2.0.0 after Tag/Release: 1.0.0 was released, note the missing v here) and commit + push
  - `Package.json` for Angular projects
  - `VERSION` file for C# projects
- [ ] Create Milestone for **new** MAJOR Version `MAJOR.0.0` (for example 8.0.0) on GitHub or GitLab
