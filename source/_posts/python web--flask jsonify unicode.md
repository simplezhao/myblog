---
title: python web--flask jsonify unicode
tags:
  - python
  - flask
categories: flask
abbrlink: 9b
date: 2021-03-31 16:10:09
keywords:
description:
---

> 类似于json.dumps中ensure_ascii的配置，在flask中也有类似配置，来避免jsonify时，返回unicode编码字符

在flask config中加入`app.config['JSON_AS_ASCII'] = False`来避免jsonify对非ascii字符进行unicode编码

```python
app = Flask(__name__)
app.config.from_object(app_config)
app.config.from_mapping(
    SQLALCHEMY_DATABASE_URI=f"postgresql+psycopg2://{app_config.DB_USER}:{app_config.DB_PASSWORD}"
                            f"@{app_config.DB_HOST}:{app_config.DB_PORT}/{app_config.DB_NAME}",
    SQLALCHEMY_TRACK_MODIFICATIONS=False,
    # SQLALCHEMY_ECHO=True
)
app.config['JSON_AS_ASCII'] = False
```



## 参考

[1] [[How to prevent Unicode representation for Latin1 characters?](https://stackoverflow.com/questions/37531067/how-to-prevent-unicode-representation-for-latin1-characters)]

