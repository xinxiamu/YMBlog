---
title: Vault学习
date: 2018-03-09 10:11:17
categories: 密码
tags: vault
---

Valut是个密码管理工具，用来安全的管理例如数据库、应用程序api等等等的密码……

特性：

1.安全的私密信息存储 

2.动态的私密信息支持

3.提供对于私密信息的更新，延长有效时间的功能

4.灵活的权限控制

5.多种客户端登录验证方式

参考：
https://www.vaultproject.io/intro/index.html

## 安装Valut

1.下载地址：https://www.vaultproject.io/downloads.html

2.解压
解压后只有一个名为vualt的可执行文件。该文件可以安全的移动位置。

配置环境变量，把vault可执行文件所在目录添加到环境变量：

`export PATH=$PATH:/home/mutian/dev/bin`

检验是否成功：
    
    mutian@mutian-ThinkPad-T440p:~$ vault 
    Usage: vault <command> [args]
    
    Common commands:
        read        Read data and retrieves secrets
        write       Write data, configuration, and secrets
        delete      Delete secrets and configuration
        list        List data or secrets
        login       Authenticate locally
        server      Start a Vault server
        status      Print seal and HA status
        unwrap      Unwrap a wrapped secret
    
    Other commands:
        audit          Interact with audit devices
        auth           Interact with auth methods
        lease          Interact with leases
        operator       Perform operator-specific tasks
        path-help      Retrieve API help for paths
        policy         Interact with policies
        secrets        Interact with secrets engines
        ssh            Initiate an SSH session
        token          Interact with tokens

表示环境变量添加准确，已经安装成功。


3.install completions

    $ vault -autocomplete-install
    
然后重新启动shell窗口，输入命令`vault`，然后按Tab键，将出现命令参数提示。如下：    

    mutian@mutian-ThinkPad-T440p:~$ vault 
    audit      lease      operator   read       ssh        unwrap     
    auth       list       path-help  secrets    status     write      
    delete     login      policy     server     token      
    mutian@mutian-ThinkPad-T440p:~$ vault 

##　启动服务

#### 启动开发环境

开发环境只用来在本机做开发使用，数据保存在内存，所以千万不能在生产环境使用。

    mutian@mutian-ThinkPad-T440p:~$ vault server -dev
    ==> Vault server configuration:
    
                         Cgo: disabled
             Cluster Address: https://127.0.0.1:8201
                  Listener 1: tcp (addr: "127.0.0.1:8200", cluster address: "127.0.0.1:8201", tls: "disabled")
                   Log Level: info
                       Mlock: supported: true, enabled: false
            Redirect Address: http://127.0.0.1:8200
                     Storage: inmem
                     Version: Vault v0.9.5
                 Version Sha: 36edb4d42380d89a897e7f633046423240b710d9
    
    WARNING! dev mode is enabled! In this mode, Vault runs entirely in-memory
    and starts unsealed with a single unseal key. The root token is already
    authenticated to the CLI, so you can immediately begin using Vault.
    
    You may need to set the following environment variable:
    
        $ export VAULT_ADDR='http://127.0.0.1:8200'
    
    The unseal key and root token are displayed below in case you want to
    seal/unseal the Vault or re-authenticate.
    
    Unseal Key: CHZrUesD0FeIHV/5lzkeKehzYh+pNjd0GH5wzG0VjSE=
    Root Token: 182a4bb0-1165-a049-6e7a-e0dbef229a28
    
    Development mode should NOT be used in production installations!
    
    ==> Vault server started! Log data will stream in below:

看到上面内容说明已经启动成功，在前台运行的。

验证服务是否在成功运行：

    mutian@mutian-ThinkPad-T440p:~$ vault status 
    Error checking seal status: Get https://127.0.0.1:8200/v1/sys/seal-status: http: server gave HTTP response to HTTPS client
    mutian@mutian-ThinkPad-T440p:~$ 

看到以上提示：错误，所以启动没成功。因为没配置回环访问。执行如下命令：

     export VAULT_ADDR=http://127.0.0.1:8200
     
再次查看vault服务运行状态:

    mutian@mutian-ThinkPad-T440p:~$ vault status
    Key             Value
    ---             -----
    Seal Type       shamir
    Sealed          false
    Total Shares    1
    Threshold       1
    Version         0.9.5
    Cluster Name    vault-cluster-bcf3f2f8
    Cluster ID      c2649684-fe35-3820-983b-f324a51b115c
    HA Enabled      false
         
看到了上面的内容，则证明服务启动成功。

为了方便CLI使用vault命令，建议配置环境变量；安全起见，建议设置环境变量只在当前客户端生效， 
命令：

| 功能      | 命令    |  说明  |
| --------   | -----   | :---- |
| 设置vault访问地址       | export VAULT_ADDR=http://127.0.0.1:8200     |   vault命令作用的vault服务的地址    |
| 设置Vault PATH        | export PATH=$PATH:< vault install path >      |   vault install path：vault安装路径    |
| 设置访问token        | 	export VAULT_TOKEN=< token >      |   token：登录vault时的token，首次登录可使用root token    |
| 设置是否跳过核查        | 	export VAULT_SKIP_VERIFY=false      |    使用TSL访问时需要设置，未使用证书忽略此项    |
| 设置访问证书        | 	export VAULT_CAPATH=/usr/local/vault/work/ca/certs/ca.cert.pem      |   使用TSL访问时需要设置，未使用证书忽略此项    |

     
- 保存私密信息

下面是简单的写入信息命令：

    mutian@mutian-ThinkPad-T440p:~$ vault write secret/hello value=world
    Success! Data written to: secret/hello

这会把键值对信息写入到路劲`secret/hello`中。　键为value,值为world。

也可以一次性写入多个键值保存：

    mutian@mutian-ThinkPad-T440p:~$ vault write secret/hello value=world excited=yesSuccess! Data written to: secret/hello

- 读取私密信息
    
显示该路径下所有保存键值对：

    mutian@mutian-ThinkPad-T440p:~$ vault read secret/hello 
    Key                 Value
    ---                 -----
    refresh_interval    768h
    excited             yes
    value               world    

获取单个的值：

    mutian@mutian-ThinkPad-T440p:~$ vault read -field value secret/hello
    world

- 删除路径下键值

删除所有：

    mutian@mutian-ThinkPad-T440p:~$ vault delete secret/hello 
    Success! Data deleted (if it existed) at: secret/hello
    mutian@mutian-ThinkPad-T440p:~$ vault read secret/hello
    No value found at secret/hello

看上面命令，说明已经把保存到路径`secret/hello`下的键值信息全部删除。

## 秘密引擎

上面内容中，我们知道怎么保存信息，读取信息，删除信息，但是注意到没，只能保存到路径`secret/hello`下面，这个是默认的。当你试图保存到其他路径下时候，将报错。

    mutian@mutian-ThinkPad-T440p:~$ vault write ~/dev name=zmt
    Error writing data to home/mutian/dev: Error making API request.
    
    URL: PUT http://127.0.0.1:8200/v1/home/mutian/dev
    Code: 404. Errors:
    
    * no handler for route 'home/mutian/dev'

默认下，在路劲`secret/.`Vault开启一个ｋｖ引擎。这个ｋｖ引擎可写入，读取数据到后台存储。    

- 开启一个新的kv私密引擎

    mutian@mutian-ThinkPad-T440p:~$ vault secrets enable -path=abs kv
    Success! Enabled the kv secrets engine at: abs/
    mutian@mutian-ThinkPad-T440p:~$ vault secrets list
    Path          Type         Description
    ----          ----         -----------
    abs/          kv           n/a
    cubbyhole/    cubbyhole    per-token private secret storage
    identity/     identity     identity store
    secret/       kv           key/value secret storage
    sys/          system       system endpoints used for control, policy and debugging

通过命令`vault secrets list`可以看到，第一个就是就是我们刚才开启的新私密引擎。

往该新建私密引擎保存私密信息：

    mutian@mutian-ThinkPad-T440p:~$ vault write abs/my-secret name=zmt
    Success! Data written to: abs/my-secret
    mutian@mutian-ThinkPad-T440p:~$ vault write abs/hello target=world
    Success! Data written to: abs/hello
    mutian@mutian-ThinkPad-T440p:~$ vault write abs/airplane type=boeing class=787
    Success! Data written to: abs/airplane
    mutian@mutian-ThinkPad-T440p:~$ 
    
查看该私密引擎下所有key

    mutian@mutian-ThinkPad-T440p:~$ vault list abs
    Keys
    ----
    airplane
    hello
    my-secret

- 关闭私密引擎

当一个私密引擎不再使用的话，我们就可以调用命令来停用它。　　
当停用一个私密引擎的时候，该私密引擎将撤销，对应的保存的私密信息将会被移除。

    mutian@mutian-ThinkPad-T440p:~$ vault secrets disable abs/
    Success! Disabled the secrets engine (if it existed) at: abs/
    mutian@mutian-ThinkPad-T440p:~$ vault secrets list
    Path          Type         Description
    ----          ----         -----------
    cubbyhole/    cubbyhole    per-token private secret storage
    identity/     identity     identity store
    secret/       kv           key/value secret storage
    sys/          system       system endpoints used for control, policy and debugging
    
上面结果中，已经再看不到私密引擎`abs/`

- 什么是私密引擎（Secrets Engine）

上面我们学会了如何启动停止一个私密引擎，那私密引擎到底是个什么东西呢？

实际上，私密引擎就类似一个虚拟文件系统，所有的read/write/delete/list操作都在它下面进行，然后私密引擎自己决定如何来响应请求。　
这是一种抽象，这种抽象具有强大的作用，它提供统一的接口，直接面对物理系统、数据库等，除此之外，一些独特的环境如AWS IAM、动态sql等，都可以统一使用增删改查这些操作接口。　


## 动态私密信息保存



    
