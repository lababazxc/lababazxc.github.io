---
title: “pytest在多线程运行测试用的一个问题”
date: 2022-07-26 16:40:02
tags:
---

### 问题描述
在自动化测试实践中，往往用例数量会达到数千条。如果采用单线程运行的话，执行效率就会十分低下。假设一条用例执行需要1s，共有2000条用例，则执行一次所需时间达到30分钟。更不用说在大数据业务场景中，绝大多数用例都涉及提交任务运行、计算的场景，时间会被进一步拉长。这在ci环节对用户是十分不友好的。因此引入了**pytest-xdist**这个插件来辅助用例执行。但是实际应用中会产生一些问题。

在做自动化测试的时候，通常我们会将登陆接口--login，放到fixture中并且将scope设置为session，让登陆操作全局只运行一次。但是当我们使用了pytest-xdist的时候，用例执行时程序会启动多个线程执，此时login并非只执行了一次，而是与线程数一致。即n个线程就会执行n遍login。这有违使用fixture的初衷

---

### 解决方案
pytest-xdist官方给出了一段示例来解决上述问题, 下面以登陆为例对这段代码进行说明
```
import json

import pytest
from filelock import FileLock

@pytest.fixture(scope="session")
def session_data(tmp_path_factory, worker_id):
    if worker_id == "master":
        # not executing in with multiple workers, just produce the data and let
        # pytest's fixture caching do its job
        return login()

    # get the temp directory shared by all workers
    root_tmp_dir = tmp_path_factory.getbasetemp().parent

    fn = root_tmp_dir / "data.json"
    with FileLock(str(fn) + ".lock"):
        if fn.is_file():
            token = json.loads(fn.read_text())
        else:
            token = login()
            fn.write_text(json.dumps(token))
    return data

```
第一段if判断的是是否为多进程运行。多线程运行状态下，xdist会创建一个master和多个worker。master负责分配用例给worker执行。分配策略这里不过多赘述。如果`worker_id`为master，说明此时为单机运行，直接返回登陆后的token。

剩下的代码段都是在多线程下的执行逻辑。`fn`为一个json文件，文件内容为登陆后获得的token。
在线程获取到锁后：如果文件不存在，即为第一次登陆。执行登陆逻辑，并将token写入文件。如果文件存在，则直接从文件中读取token。
如此通过文件锁的方式，保证了登陆逻辑只执行一次。

