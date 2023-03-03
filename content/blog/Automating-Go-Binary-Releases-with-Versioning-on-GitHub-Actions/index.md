---
author: Daniel Ola-Olorun
categories:
- Github Actions
- Golang
- Automation
date: "2023-03-02"
draft: true
excerpt: 
layout: single
subtitle: 
title: Automating Go Binary Releases with Versioning on GitHub Actions
---
 #### INTRODUCTION 


 Automating the release process for your Go project can save time, reduce errors, and ensure that releases are consistent across different environments. In this blog post, we'll look at how to automate the release of Go binaries with versioning on GitHub Actions.

###### Prerequisites

Before we begin, you should have the following set up:

A Go project with a build script that generates a binary.

A GitHub repository for the project.

A basic understanding of GitHub Actions.

Step 1: Create a Release Workflow

The first step is to create a workflow that will run when a new release is created. In your repository, create a new file in the .github/workflows directory called release.yml.

name: Release

on:
  release:
    types: [created]

This workflow will trigger when a new release is created in the repository.

Step 2: Set Up the Build Environment

In this step, we'll set up the build environment for our Go project. We'll use the official Go Docker image to build the project.

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Set up Go
        uses: actions/setup-go@v2
        with:
          go-version: 1.16
          
      - name: Build
        run: go build -o myapp main.go


This workflow defines a job called build that runs on Ubuntu. It checks out the repository, sets up Go 1.16, and builds the project.

Step 3: Publish the Release

Now that we have built the binary, we need to publish it as part of the release. We'll use the official @actions/upload-release-asset action to upload the binary to the release.


jobs:
  build:
    # ...
    
    steps:
      # ...
      
      - name: Publish Release
        id: release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          body: |
            Changes in this release:
            - foo
            - bar
          draft: false
          prerelease: false

      - name: Upload Binary
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.release.outputs.upload_url }}
          asset_path: ./myapp
          asset_name: myapp-${{ github.ref }}-${{ matrix.os }}
          asset_content_type: application/octet-stream

This workflow uses the actions/create-release action to create a new release with a tag and release name based on the Git tag. It also sets the release notes and disables the draft and pre-release options.

Next, the actions/upload-release-asset action is used to upload the binary to the release. We specify the upload URL from the previous step and provide a name for the binary that includes the Git tag and the operating system being used.

Step 4: Add Versioning to the Binary

To ensure that the binary is versioned, we'll add the version number to the binary's filename. We'll use the ldflags feature of the Go build command to set the version number.


go build -ldflags "-X main.Version=$(git describe --tags --always --dirty)" -o myapp main.go
``

Step 5: Update the Workflow to Include Versioning

Now that we've set up versioning for our binary, we need to update the workflow to use the new binary filename.


jobs:
  build:
    # ...
    
    steps:
      # ...

      - name: Set Version
        run: |
          echo ::set-env name=BINARY_NAME::myapp-$(git describe --tags --always --dirty)
          echo "Binary name: $BINARY_NAME"

      - name: Upload Binary
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.release.outputs.upload_url }}
          asset_path: ./myapp
          asset_name: ${{ env.BINARY_NAME }}
          asset_content_type: application/octet-stream



In this updated workflow, we use the echo command to set the BINARY_NAME environment variable to the new binary filename, which includes the Git tag and the operating system being used. We then use this variable when uploading the binary to the release.

Step 6: Test the Workflow

Finally, we can test the workflow by creating a new release in our repository. When the release is created, the workflow will trigger and build the binary, set the version number, and upload the binary to the release.

Conclusion

In this blog post, we looked at how to automate the release of Go binaries with versioning on GitHub Actions. By automating this process, we can save time and ensure that our releases are consistent across different environments. By setting up versioning for our binaries, we can easily track changes and roll back if necessary.
