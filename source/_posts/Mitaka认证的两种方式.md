---
layout: 获取openstack
title: Mitaka认证的两种方式
date: 2016-04-05 04:56:12
tags:
 - OpenStack
---

## 背景
    随着OpenStack不断发展, 目前的MITAKA版本的keystone已经弃用了第二版本的身份认证API, 而且我在M版本下使用原来版本的keystoneclient时, 发现提示如下
    
>  DeprecationWarning: keystoneclient auth plugins are deprecated as of the 2.1.0 release in favor of keystoneauth1 plugins. They will be removed in future releases.

    因此, 在Mitaka版本中使用keystoneclient已不可取, 而且第三版的身份认证API和第二代有比较大的不同.

-------------
## Keystone第三版身份认证相关参数
只涉及到
>`project_domain`: 项目域
>`project_name`:项目(租户)
>`user_domain_name`:用户域
>`username`:用户名
>`password`:密码
>`auth_url`:认证地址, "**http://controller:5000/v3/**"
> region: 区域, Regions之间完全隔离，但它们共享同一个Keystone和Dashboard服务
> auth_plugin: 认证方式, "**password**"
>其中, 高亮部分为必须参数,其余为可选参数

----------------
## 在代码中直接指定相关参数

```python
def create_connection(project_domain_name, user_domain_name, auth_url, region, project_name, username, password):
    prof = profile.Profile()
    prof.set_region(profile.Profile.ALL, region)

    return connection.Connection(
        profile=prof,
        user_agent='Autumn',
        project_domain_name=project_domain_name,
        user_domain_name=user_domain_name,
        auth_url=auth_url,
        project_name=project_name,
        username=username,
        password=password
    )

def list_images(conn):
    print("List Images:")

    for image in conn.image.images():
        print(image)

if __name__ == '__main__':

    auth_args = {
        'project_domain_name':'default',
        'user_domain_name':'default',
        'auth_url': 'http://127.0.0.1:5000/v3',
        'region':'RegionOne',
        'project_name': 'admin',
        'username': 'admin',
        'password': 'password',
    }
    conn = create_connection(**auth_args)
    list_images(conn)
```

------------------
## 从文件中给定相关参数

 新建cloud.yaml, 并加载cloud.ymal
> ```
> $ export OS_CLIENT_CONFIG_FILE=/yourPath/clouds.yaml
> ```

```json
cloud.yaml

clouds:
  test_cloud:
    region_name: RegionOne
    auth:
      auth_url: http://127.0.0.1:5000/v3/
      username: admin
      password: password
      project_name: admin
      domain_name: default
```

```python
TEST_CLOUD = os.getenv('OS_TEST_CLOUD', 'test_cloud')
class Opts(object):
    def __init__(self, cloud_name='test_cloud', debug=False):
        self.cloud = cloud_name
        self.debug = debug
        self.identity_api_version = '3'
opts = Opts(cloud_name=TEST_CLOUD)
occ = os_client_config.OpenStackConfig()
cloud = occ.get_one_cloud(opts.cloud, argparse=opts)

def list_images(conn):
    print("List Images:")
    for image in conn.image.images():
        print(image)
        
    list_images(connection.from_config(cloud_config=cloud, options=opts))
```
