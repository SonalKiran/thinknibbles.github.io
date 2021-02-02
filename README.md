# Read me

## Must do
- Please create a branch with your own name and do all the edits in that branch only. Once the edits are finalised, then only merge to main / master branch

## Pre - requisites

- Refer to [this page](https://jekyllrb.com/docs/installation/) for installing pre requisites
- Run the following command from the root folder ```bundle install```

## How to add posts

- browse inside the '_posts' folder
- Add a new markdown file and make sure following values are according to your use case ( refer to anyone of the existing MD files for an example )
  - layout: post
  - title:  "This contains the title of this post"
  - date:   2020-12-06 00:34:42 +0800
  - categories: [samplecategory]
  - tag: [sampletag]



## How to run server locally to view github pages locally

- browse to root folder of this project and run the following command

  ```bundle exec jekyll serve --livereload```

- This will create a jekyll server locally and allow you to access the github pages on http://localhost:4000/
