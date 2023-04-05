# Code Review

<!-- markdown-toc Marked -->

* [在Gitlab中配置DangerJS](#在gitlab中配置dangerjs)
  * [机器人账号](#机器人账号)
  * [DangerJS初始化](#dangerjs初始化)
  * [项目设置](#项目设置)
  * [配置CI](#配置ci)
  * [本地调用CI运行](#本地调用ci运行)
  * [最佳实践](#最佳实践)

<!-- markdown-toc -->

## 在Gitlab中配置DangerJS

### 机器人账号

建议使用专门的账号作为机器人账号，用于发表DangerJS校验评论提醒。

在个人设置创建一个名为`DANGER_GITLAB_API_TOKEN`的`Access Token`，在创建过程中需要设置过期时间及权限范围。

1. 过期时间：对于专门的机器人账号可以选择较长的时间。
2. 权限范围：由于需要读&写两种权限，所以选择`api`。

在创建`DANGER_GITLAB_API_TOKEN`完成后会显示对应的token值，务必将这个token值记录下来备用，因为只会显示一次！！！

### DangerJS初始化

在项目中安装依赖
```bash
npm install -D danger
```

在项目根目录中创建`dangerfile.{js|ts}`文件，在其中写入想要让DangerJS帮助我们完成的工作。这里简单判断MR状态:
```javascript
import { danger, warn } from "danger"

if (danger.gitlab.mr.title.includes("Draft")) {
  warn("This is a draft MR")
}
```

同时，可通过npm命令向`package.json`中添加对应的命令

```bash
npm set-script 'ci:danger' 'npx danger ci --failOnErrors -v'
```

### 项目设置

项目中需要做三个操作

1. 添加机器人账号到项目成员中。
2. 设置Gitlab环境变量，具体路径为`Settings` > `CI/CD` > `Variables`中，添加的变量如下：
  + 变量名为`DANGER_GITLAB_API_TOKEN`，值为刚才创建的机器人账号的Access Token - `DANGER_GITLAB_API_TOKEN`
  + 变量名为`DANGER_GITLAB_HOST`，值为`http://git.dev.sh.ctripcorp.com`，对应公司Gitlab的host
3. 配置CI。 

### 配置CI

推荐在`VerifyAndBuild`前添加一个`Check`的阶段，以我们项目为例的Danger任务配置如下：

```yaml
Danger:
  stage: Check
  image: hub.cloud.ctripcorp.com/nfesres/nfes_base:nfes5_16.V2
  tags:
    - official-uat
  cache:
    key: $CI_COMMIT_REF_SLUG-$CI_COMMIT_SHORT_SHA
    paths:
      - node_modules
      - scripts
    policy: pull
  script:
    - npm run ci:danger
  only:
    - merge_requests
```

至此项目的配置基本完成，可以通过创建一个MR来测试效果。

### 本地调用CI运行

如果想要在本地调试`dangerfile.{js|ts}`文件，可以通过danger提供的本地调试功能完成。具体操作如下：

1. 在命令行中设置本地环境变量
  + `export DANGER_GITLAB_API_TOKEN={机器人账号AccessToken}`
  + `export DANGER_GITLAB_HOST=http://git.dev.sh.ctripcorp.com`
2. 准备好Merge Request URL，如`http://git.dev.sh.ctripcorp.com/FlightMobile/aws-lighthouse-site/-/merge_requests/252`
3. 在命令行中运行 `npx danger pr {MergeRequestURL}`

### 最佳实践

将Merge Request Template和Dangerfile相互配合，实现一系列自动化检测的功能。

<details>
  <summary style="cursor: pointer; text-decoration:underline; color: #2AD;">Merge Request Template</summary>

```markdown
# Ticket

# Why has this change been made?

# What changed?

  * changes

# Screenshot

  <img width="400" alt="image" src="#">

## For all changes

  - [ ] Code has been reviewed by the author
  - [ ] Manually tested in a web browser
  - [ ] Automated tests added/updated

## For UI changes

  - [ ] Manually tested on a real mobile device
  - [ ] Manually tested in other browsers (which ones?)
  - [ ] Manually tested RTL support (with screenshot)
  - [ ] Manually tested accessibility in main browsers: Chrome, Firefox and Safari

# Danger toggles

  - [ ] Skip MR size check
  - [ ] I am familiar with the danger of releasing webapp and server changes simultaneously. I agree that the changes here are backwards compatible and that this MR does not need to be split.
```
</details>


<details>
  <summary style="cursor: pointer; text-decoration:underline; color: #2AD;">Dangerfile</summary>

```javascript
import { danger } from 'danger';
const { commonMrDescription, mrDescription, inCommitGrep } = require('./danger/tools');
const {
  git: { created_files: createdFiles, modified_files: modifiedFiles },
  gitlab: {
    utils: { addLabels }
  }
} = danger;
 
const fileChanges = [...modifiedFiles, ...createdFiles];

/**
 * Check there is a description for internal or external contributions
 */
commonMrDescription({
  minLength: 50,
  logType: 'message',
  msg: 'Please provide more context about your MR that other engineers, or your future self, would find useful.'
});

/**
 * Hard fail if MR author has not reviewed their code
 */
const MR_SELF_REVIEW_CHECK = 'Code has been reviewed by the author';
const hasPrAuthorSelfReviewed = mrDescription.includes(`[x] ${MR_SELF_REVIEW_CHECK}`);
if (!hasPrAuthorSelfReviewed) {
  fail(
    `\`${MR_SELF_REVIEW_CHECK}\` is unchecked in the MR description.\n\n` +
      "* Please ensure you have read through your code changes in 'Files changed' _before_ asking others to review your MR. " +
      "This helps catch obvious errors and typos, which avoids wasting your time and the reviewer's time, and ultimately allows us to " +
      'ship product changes more quickly.\n\n* Please also take this opportunity to annotate your MR with GitHub comments to explain ' +
      'any complex logic or decisions that you have made. This helps the reviewer understand your MR and speed up the review process.\n\n' +
      `Once you've done this, check \`${MR_SELF_REVIEW_CHECK}\` in the MR description to make this Danger step pass.`
  );
}

/**
 * Hard fail if MR author has not manually tested their changes when source files have been modified
 */
const MANUAL_TEST_CHECK = 'Manually tested in a web browser';
const hasBeenManuallyTested = mrDescription.includes(`[x] ${MANUAL_TEST_CHECK}`);
const hasSourceFileChanges = inCommitGrep(/packages\/(client|server)\/.*.(?<!test.)(js|ts|jsx|tsx)$/);
if (hasSourceFileChanges && !hasBeenManuallyTested) {
  fail(
    `\`${MANUAL_TEST_CHECK}\` is unchecked in the MR description when source files have been modified.\n\n` +
      '* Please ensure you have manually tested your change in a web browser.\n\n* Preferably, also manually test the full search flow.\n\n' +
      '* This helps prevent bugs that automated tests miss from reaching production. Ultimately, this saves travellers from ' +
      'suffering a degraded experience and saves us from wasting time in incidents/ILDs that could be avoided.\n\n' +
      "* Manual testing is _not_ a substitute for automated testing - if you've made logic changes, please ensure they are covered by tests.\n\n" +
      `Once you've done this, check \`${MANUAL_TEST_CHECK}\` in the MR description to make this Danger step pass.`
  );
}

/**
 * Hard fail if source files have been modified, but not test files
 */
const AUTOMATED_TEST_CHECK = 'Automated tests added/updated';
const hasUpdatedAutomatedTests = mrDescription.includes(`[x] ${AUTOMATED_TEST_CHECK}`);
const hasTestFileChanges = inCommitGrep(/packages\/(client|server)\/.*test.*(js|ts|jsx|tsx)$/);
if (hasSourceFileChanges && !hasTestFileChanges && !hasUpdatedAutomatedTests) {
  fail(
    'Source files have been modified, but no test files have been added or modified.\n\n' +
      "If you've made logic changes, please ensure they are covered by automated tests.\n\n" +
      `Once you've done this, check \`${AUTOMATED_TEST_CHECK}\` in the MR description to make this Danger step pass.`
  );
}

/**
 * Encourage smaller MRs.
 */
const SKIP_BIG_MR_FAIL_CHECK = 'Skip MR size check';
const SKIP_BIG_MR_FAIL = mrDescription.includes(`[x] ${SKIP_BIG_MR_FAIL_CHECK}`);
const bigMRThreshold = 10;
if (!SKIP_BIG_MR_FAIL && fileChanges.length > bigMRThreshold) {
  fail(
    `This MR contains ${fileChanges.length} files (${createdFiles.length} new, ${modifiedFiles.length} modified). Consider splitting it into multiple MRs. Otherwise toggle the danger check '[ ] Skip MR size check' in the MR template`
  );
}

/**
 * Labelled MRs.
 */
const MR_LABELS = [
  {
    match: inCommitGrep(/packages\/server\/(?!app\/tests\/).*/),
    name: 'server'
  },
  {
    match: inCommitGrep(/packages\/client\/.*/),
    name: 'webapp'
  }
];

MR_LABELS.forEach(({ match, name }) => {
  if (match) {
    addLabels(name);
  }
});

const SKIP_SERVER_CLIENT_FAIL_CHECK_REVIEWER =
  'I am familiar with the danger of releasing webapp and server changes simultaneously';

const SKIP_SERVER_CLIENT_REVIEWER_FAIL = mrDescription.includes(`[x] ${SKIP_SERVER_CLIENT_FAIL_CHECK_REVIEWER}`);

/**
 * Check if changes have been made to both the server and client code
 */
if (inCommitGrep(/packages\/client\/.*/) && inCommitGrep(/packages\/server\/(?!app\/tests\/).*/)) {
  if (!SKIP_SERVER_CLIENT_REVIEWER_FAIL) {
    fail(
      `The reviewer must check the \`${SKIP_SERVER_CLIENT_FAIL_CHECK_REVIEWER}\` toggle, or the pull request should be split.`
    );
  } else {
    warn(
      'Please be extra careful when making changes to code in both the webapp and server code.\n' +
        'Any API changes need to be deployed ahead of client changes as not all instances will be instantly updated.'
    );
  }
}
```
</details>
