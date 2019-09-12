---
title: 使用Dock搭建Sentry服务
date: 2019-09-12 16:55:21
tags:
---

## 搭建环境

- CentOS 7.5 64 位
- docker: 18.09.6
- docker-compose: 1.24.0
- sentry: [sentry 9.1-onbuild](https://hub.docker.com/_/sentry)

## 搭建步骤

1. 安装 `docker`、`docker-compose`、`git`
2. 执行 `systemctl start docker` 启动 docker。

   ```bash
   $ systemctl start docker
   ```

   ⚡️: **可执行 `systemctl enable docker` 将启动 docker 加入开机自启**

   ```bash
   $ systemctl enable docker
   ```

3. 克隆 `sentry`到本地。`git clone https://github.com/getsentry/onpremise.git`。

   ```bash
   $ git clone https://github.com/getsentry/onpremise.git
   ```

4. 执行 `cd onpremise` 进入 onpremise 文件夹。

5. 执行 `docker volume create --name=sentry-data && docker volume create --name=sentry-postgres`

   ```docker
   $ docker volume create --name=sentry-data && docker volume create --name=sentry-postgres
   ```

   ⚡️: **使用 `docker volume` 必须手动创建本地数据库和 sentry 卷。**

6. 执行 `cp -n .env.example .env` 创建 `.env` 文件。

   ```bash
   $ cp -n .env.example .env
   ```

7. 执行 `docker-compose build` 构建并标记 `docker` 服务。

   ```bash
   $ docker-compose build
   ```

8. 执行 `docker-compose run --rm web config generate-secret-key` - 生成密钥。将它添加到 `.env` 作为 `SENTRY_SECRET_KEY`的值，还要将其添加到 `docker-compose.yml`中。

   ```bash
   $ docker-compose run --rm web config generate-secret-key
   ```

   ![.env](1.png)

   ![docker-compose.yml](2.png)

9. 执行 `docker-compose run --rm web upgrade` - 构建数据库。

   ```bash
   $ docker-compose run --rm web upgrade
   ```

   ⚡️: **官方文档 说这里会 使用交互式提示创建用户帐户 。在实际操作中却没有提示。::如果没有提示需要执行第 10 步::。**

10. 执行 `docker exec -it onpremise_postgres_1 bash` 进入 docker 容器 执行 **postgres bash** 命令查看是否有数据。

    ```bash
    $ docker exec -it onpremise_postgres_1 bash
    ```

    ![postgres bash](3.png)

    1. 执行 `psql -h 127.0.0.1 -d postgres -U postgres` 进入 postgres 数据库

    ```sql
    $ psql -h 127.0.0.1 -d postgres -U postgres
    ```

    2. 执行 `select * from sentry_project;` 查看 sentry_project 表是否有数据。

    ```sql
    $ select * from sentry_project;
    ```

    3. 执行 `select * from sentry_organization;` 查看 sentry_organization 表是否有数据。

    ```sql
    $ select * from sentry_organization;
    ```

    4. 执行 `ctrl + d` 退出 shell。

11. 如果没有数据需要添加， 执行 `docker-compose run --rm web shell` 进入 sentry 的 web 的 shell 里面。初始化数据

    ```bash
    $ docker-compose run --rm web shell
    ```

    1. 执行 `from sentry.models import Project`。

    ```sql
    $ from sentry.models import Project
    ```

    2. 执行 `from sentry.receivers.core import create_default_projects`。

    ```sql
    $ from sentry.receivers.core import create_default_projects
    ```

    3. 执行 `create_default_projects([Project])`。

    ```sql
    $ create_default_projects([Project])
    ```

    4. 执行 `ctrl + d` 退出。

    ![创建项目](4.png)

12. 执行 `docker-compose run --rm web createuser` 创建用户。

    ```bash
    docker-compose run --rm web createuser
    ```

    ![创建用户](5.png)

13. 执行 `docker-compose up -d` - 构建启动容器。
14. 在浏览器中输入 `[ip]:9000`。如果没有错误，就应该能看到了。

## 安装碰到的问题

- 💥 在执行 `docker-compose run --rm web upgrade` 的时候。可能会出现没有执行完就退出了终端。

  ![没有执行完成](6.png)

  🔨 解决方案: 需要重新执行上面的命令

  ![正常执行完成](7.png)

- 💥 登录到项目后点击 Create a sample event 测试时， 会发现是失败的，而且这个时候在项目中产生的错误不会在这个 Issues 列表中展示。

  ![点击测试无效](8.png)

  🔨 解决方案:

  参考：[Waiting for events… Our error robot is waiting to devour receive your first event - #sentry](https://forum.sentry.io/t/waiting-for-events-our-error-robot-is-waiting-to-devour-receive-your-first-event/4355)

  1. 执行 `docker exec -it onpremise_postgres_1 bash` 进入 docker 容器 执行 **postgres bash** 命令查看是否有数据。

  ```bash
  $ docker exec -it onpremise_postgres_1 bash
  ```

  2. 执行 `psql -h 127.0.0.1 -d postgres -U postgres` 进入 postgres 数据库

  ```bash
  $ psql -h 127.0.0.1 -d postgres -U postgres
  ```

  3. 执行下面 SQL 语句后。在浏览器中点击 Create a sample event 就好了，也正常记录 Issue 了。

  ```sql
  create or replace function sentry_increment_project_counter( project bigint, delta int) returns int as $$ declare new_val int;
  begin loop update sentry_projectcounter set value = value + delta where project_id = project returning value into new_val;
  if found then return new_val;
  end if;
  begin insert into sentry_projectcounter(project_id, value) values (project, delta) returning value into new_val;
  return new_val;
  exception when unique_violation then end;
  end loop;
  end $$ language plpgsql;
  ```
