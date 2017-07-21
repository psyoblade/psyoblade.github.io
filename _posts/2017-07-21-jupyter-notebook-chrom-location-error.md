---
layout: post
title:  "‘ open location’ 메시지를 인식하지 못합니다. (-1708)"
date:   2017-07-21 23:15:22 +0900
categories: psyoblade update
---
## 최신 버전 맥북에서 주피터 노트북을 콘솔에서 실행 시에 아래와 같은 오류가 뜨면서 실행되지 않는경우
execution error: "http://localhost:8888/tree?token=deaf57dcb77644fc0362cb128b83aa357dbc78ed84c6c98f"이(가) ‘ open location’ 메시지를 인식하지 못합니다. (-1708)"
```
    jupyter notebook --generate-config
    vi ~/.jupyter/.jupyter_notebook_config.py
    c.NotebookApp.browser = u'chrome'
```

