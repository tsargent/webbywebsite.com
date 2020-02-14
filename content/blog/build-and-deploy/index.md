---
title: Build and Deploy
date: "2020-02-02"
description: "Day #2 of 100 Days of Gatsby"
image: builders.jpg
---
This post covers my solution to the second challenge of [100 Days of Gatsby](https://www.gatsbyjs.org/blog/100days/). The goal was to get the site to a host, while using free tools. More details on this challenge [here](https://www.gatsbyjs.org/blog/100days/free-hosting/).

I chose to recycle some work that I had already done for a site that I had not yet really built out. A few months ago, I went through the fantastic [AWS for Front-End Engineers](https://frontendmasters.com/courses/aws-frontend-react/) course on [Frontend Masters](https://frontendmasters.com/) to learn how to utilize AWS tools for this kind of task. Since I already had that pipeline set up, and I wasn't really using it for anything, I chose to just deliver this site through that pipeline, and explain that setup here. For other projects I have used [Netflify](https://www.netlify.com/), which is an amazingly simple solution.

I won't go too far into the details of setting up all the AWS tools, as there is plenty of information out there for this kind of thing. I will just breifly outline the pipeline to explain what is going on at each step.

## GitHub
This first part is somewhat obvious, but in order to kick of an automatic deployment process, it's best to have our code in a repository on the internet. That way, we can use a service that is notified when our code is updated in someway. For this site, we use [Travis CI](https://travis-ci.com/).

## Travis CI
To use Travis CI, we sign up for an account and add our GitHub repo. Then it's mostly just a matter of adding a `.travis.yml` file to the root of the repo. This file consists of a set of instructions for Travis CI. As of this writing, mine looks like this:

```
language: node_js
node_js:
  - 10
cache:
  yarn: true
  directories:
    - node_modules
script:
  - yarn test
before_deploy:
  - yarn global add travis-ci-cloudfront-invalidation
  - yarn build
deploy:
  provider: s3
  access_key_id: $AWS_ACCESS_KEY_ID
  secret_access_key: $AWS_SECRET_ACCESS_KEY
  bucket: $S3_BUCKET
  skip_cleanup: true
  local-dir: public
  on:
    branch: master
after_deploy:
  - travis-ci-cloudfront-invalidation -a $AWS_ACCESS_KEY_ID -s $AWS_SECRET_ACCESS_KEY -c $CLOUDFRONT_ID -i '/*' -b $TRAVIS_BRANCH -p $TRAVIS_PULL_REQUEST

```

Most of this came from that Frontend Masters course I mentioned above. Let's go through it section by section:

- `language` We set the language to `node_js` because that's what Travis CI will use to run these steps.
- `node_js` Set the Node JS version to 10. I should probably update this to 12, which is now the LTS version of Node JS.
- `cache` We cache the node_modules directory, so that if our dependencies don't change between builds, Travis CI doesn't reinstall them. This can be quite time consuming, and we want to avoid it whenever possible.
- `script` We tell Travis CI to run the test script. Right now this repo doesn't have any tests. That's no good, and I will add them later. For now, this script will run and exit with exit code 0, which just means success, as if all our tests passed. 
- `before_deploy` Before Travis CI runs our deployment script, we want to add one more dependency that will be used to invalidate our CloudFront distribution. We install it here without including it as a dev dependency because we only need it for deployment. Then we run our build script, because in order to deploy the site, we need to build it. So we do `yarn build`. If we look in `package.json`, we can see that this script just runs `gatsby build`, which will build our site into the `/public` directory.
- `deploy` We deploy to an S3 bucket with the given credentials and bucket name. These values are entered into the repo settings through the Travis CI ui. Everything in the `public` directory will be deployed. This will happen when the `master` branch of our repo is updated.
- `after_deploy` Finally, we run the `travis-ci-cloudfront-invalidation` command. This invalidates our CloudFront distribution for this site, which means that instead of getting the content from the closest CloudFront edge location, it is requested from our S3 bucket, because the content is presumably new and we don't want the cached version from CloudFront.

That's a very brief overview of how deployment works for this site. Again, almost all of this comes from the  [AWS for Front-End Engineers](https://frontendmasters.com/courses/aws-frontend-react/) course, which I highly recommend. All of the tooling is handled within the [AWS Free Tier](https://aws.amazon.com/free), with Travis CI handling the deployment. 
