1. 准备

   - 项目使用bazel，如对bazel 不了解，请参与[bazel文档](https://docs.bazel.build/versions/4.2.2/bazel-overview.html)
   - 为项目引入[rulers_docker](https://github.com/bazelbuild/rulers_docker)

2. 编写脚本

   - 在项目跟目录下创建.bazelrc 文件，为设置bazel 构建或运行命令设置 --workspace-status-command参数，该参数接受一个可运行程序，这里指定了一个shell脚本，在构建命令执行时会运行该脚本，以便在构建镜像时动态指定镜像的tag，仓库目录等变量。

     ```shell
     #.bazelrc 内容
     build --workspace_status_command=./publish/command_status.sh
     run --workspace_status_command=./publish/command_status.sh
     ```

   - command_status.sh 脚本定义，用于生成构建镜像时需要动态指定的变量

     ```shell
     #!/bin/bash
     
     set -o errexit
     set -o nounset
     set -o pipefail
     
     branch=$(git rev-parse --abbrev-ref HEAD)
     branch=${BRANCH_OVERRIDE:-$branch}
     
     commit_id=$(git describe --tags --always --dirty)
     commit_id=${COMMIT_ID_OVERRIDE:-$commit_id}
     
     repository=$(git remote -v | tail -1 | awk -F '/' '{print $NF}' | awk -F ' ' '{print $1}' | awk -F '.' '{print $1}')
     if [ -z "$repository" ]; then
         repository=$(pwd | xargs dirname | xargs basename)
     fi
     repository=${REPOSITORY_OVERRIDE:-$repository}
     
     image_tag="${branch}-${commit_id}"
     image_tag=${IMAGE_TAG_OVERRIDE:-$image_tag}
     
     image_registry=${IMAGE_REGISTRY_OVERRIDE:-swr.cn-north-4.myhuaweicloud.com}
     image_repo=${IMAGE_REPO_OVERRIDE:-opensourceway/robot/$repository}
     
     cat <<EOF
     IMAGE_REGISTRY ${image_registry}
     IMAGE_REPO ${image_repo}
     IMAGE_TAG ${image_tag}
     IMAGE_ID ${image_registry}/${image_repo}:${image_tag}
     CODE_REPOSITORY ${repository}
     EOF
     ```

   - 定义bazel宏

     ```python
     #image.bzl
     load("@io_bazel_rules_docker//container:pull.bzl", "container_pull")
     load("@io_bazel_rules_docker//container:image.bzl", "container_image")
     load("@io_bazel_rules_docker//container:bundle.bzl", "container_bundle")
     load("@io_bazel_rules_docker//contrib:push-all.bzl", "container_push")
     load("@io_bazel_rules_docker//go:image.bzl", "go_image")
     
     def operating_system():
         container_pull(
                 name = "alpine_linux_amd64",
                 registry = "index.docker.io",
                 repository = "library/alpine",
                 digest = "sha256:69704ef328d05a9f806b6b8502915e6a0a4faa4d72018dc42343f511490daf8a",
                 tag = "3.14.2",
         )
     
     def build_plugin_image(name, plugin, **kwargs):
         build_image(
             name = name,
             base = "@alpine_linux_amd64//image",
             repository = plugin,
         )
     
     # build_image is a macro for creating :app and :image targets
     def build_image(
             name,
             app_name = "app",
             base = None,
             goarch = "amd64",
             goos = "linux",
             stamp = True,
             component = [":go_default_library"],
             **kwargs):
     
         go_image(
             name = app_name,
             base = base,
             embed = component,
             goarch = goarch,
             goos = goos,
         )
     
         container_image(
             name = name,
             base = ":" + app_name,
             stamp = stamp,
             entrypoint = ["/app/app.binary"],
             **kwargs
         )
     
     
     # push_image creates a bundle of container images and push them.
     def push_image(
             name,
             bundle_name = "bundle",
             images = None):
         container_bundle(
             name = bundle_name,
             images = images,
         )
         container_push(
             name = name,
             bundle = ":" + bundle_name,
             format = "Docker",
         )
     
     # image_tags returns a {image: target} map for each cmd or {name: target}
     # Concretely,image_tags("//checkpr:image") will output the following:
     # {
     #   "swr.ap-southeast-1.myhuaweicloud.com/opensourceway/robot/checkpr:20210203-deadbeef": //checkpr:image
     # }
     def image_tags(target):
         return {"{IMAGE_REGISTRY}/{IMAGE_REPO}:{IMAGE_TAG}": target}
     ```

   - 定义bazel ruler

     ```python
     load("@io_bazel_rules_go//go:def.bzl", "go_binary", "go_library")
     load("@you_project_dir//:image.bzl", "build_plugin_image", "push_image", "image_tags")
     load("@bazel_gazelle//:def.bzl", "gazelle")
     
     # gazelle:prefix github.com/opensourceways/robot-gitee-openeuler-review
     gazelle(name = "gazelle")
     
     build_plugin_image(
         name = "image",
         plugin = "robot-gitee-openeuler-review",
     )
     
     push_image(
         name = "push_image",
         images = image_tags(
             target = ":image",
         ),
     )
     ```

   - 使用bazel命令触发构建

     ```shell
     #只构建镜像
     bazel run --platforms=@io_bazel_rules_go//go/toolchain:linux_amd64 //:image
     #构建镜像并push到镜像仓
     bazel run --platforms=@io_bazel_rules_go//go/toolchain:linux_amd64 //:push_image
     ```

     

