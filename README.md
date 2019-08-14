# docker-aws-node-postgres

[![Docker Pulls](https://img.shields.io/docker/pulls/whatishedoing/docker-aws-node-postgres?style=for-the-badge)][site]
[![Docker Cloud Build Status](https://img.shields.io/docker/cloud/build/whatishedoing/docker-aws-node-postgres?style=for-the-badge)][site]

## 👋 Introduction

A [Docker] image designed to be used in [AWS] [CodeBuild] to build [Node.js] apps and connect to
[PostgreSQL] databases.

## 🏃‍ Usage

The following ordered setup instructions use [Terraform] to configure AWS.

### CodeBuild

Create a CodeBuild; note the `image` attribute of `environment`:

```ruby
resource "aws_codebuild_project" "api" {
  name          = "api"
  description   = "Builds an API and recreates a test database."
  build_timeout = 20
  service_role  = "${aws_iam_role.codebuild.arn}"

  artifacts {
    type                = "S3"
    location            = "${aws_s3_bucket.bucket.id}"
    packaging           = "ZIP"
    encryption_disabled = true
  }

  cache {
    type = "LOCAL"

    modes = [
      "LOCAL_DOCKER_LAYER_CACHE",
      "LOCAL_SOURCE_CACHE"
    ]
  }

  environment {
    compute_type    = "BUILD_GENERAL1_MEDIUM"
    image           = "whatishedoing/docker-aws-node-postgres"
    privileged_mode = false
    type            = "LINUX_CONTAINER"

    # Set any Postgres environment variables...
    # https://www.postgresql.org/docs/current/libpq-envars.html
    environment_variable {
      name  = "PG_HOST"
      value = "${aws_db_instance.db.address}"
    }

    environment_variable {
      name  = "S3_ARTIFACTS_BUCKET"
      value = "${aws_s3_bucket.bucket.id}"
    }
  }

  source {
    buildspec       = "buildspec.yml"
    git_clone_depth = 1
    location        = "${aws_codecommit_repository.api.clone_url_http}"
    type            = "CODECOMMIT"
  }

  logs_config {
    cloudwatch_logs {
      group_name  = "${aws_cloudwatch_log_group.codebuild.name}"
      stream_name = "${aws_cloudwatch_log_stream.codebuild.name}"
    }
  }

  tags = {
    Environment = "tools"
    Terraform   = true
  }
}
```

### Source Control

In the root of the source control repository, add a `buildspec.yml`:

```yml
version: 0.2

phases:
  install:
    commands:
      - yarn install --frozen-lockfile

  build:
    commands:
      - yarn start
```

[AWS]: https://aws.amazon.com/
[CodeBuild]: https://aws.amazon.com/codebuild/
[CodePipeline]: https://aws.amazon.com/codepipeline/
[Docker]: https://www.docker.com/
[Node.js]: https://nodejs.org/
[PostgreSQL]: https://www.postgresql.org/
[site]: https://hub.docker.com/r/whatishedoing/docker-aws-node-postgres
[Terraform]: https://www.terraform.io/
