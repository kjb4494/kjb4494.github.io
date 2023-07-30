## Quick Start

Before starting, please follow the instructions in the [Jekyll Docs](https://jekyllrb.com/docs/installation/) to complete the installation of `Ruby`, `RubyGems`, `Jekyll`, and `Bundler`. In addition, [Git](https://git-scm.com/) is also required to be installed.

### Installing Dependencies

Before running for the first time, go to the root directory of your site, and install dependencies as follows:

```console
$ bundle
```

### Running Local Server

Run the following command in the root directory of the site:

```console
$ bundle exec jekyll s
```

After a while, navigate to the site at <http://localhost:4000>.


### History
- Ruby 버전 최신화에 의한 구버전 코드 미작동 문제
    ```text
    https://github.com/cotes2020/jekyll-theme-chirpy
    설정 yml과 _posts를 제외한 모든 코드 최신화
    github workflows yml 수정
    ```
- `internal script /assets/js/dist/page.min.js does not exist` 문제 해결
    ```shell
    # https://github.com/cotes2020/jekyll-theme-chirpy/issues/1090
    npm install
    NODE_ENV=production npx rollup -c --bundleConfigAsCjs
    ```
- Github Ation `Your bundle only supports platforms ["x64-mingw-ucrt"] but your local platform` 문제 해결
    ```shell
    bundle install
    bundle lock --add-platform x86_64-linux
    ```
