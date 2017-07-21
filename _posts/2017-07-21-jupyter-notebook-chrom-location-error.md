## 최신 버전 맥북에서 주피터 노트북을 콘솔에서 실행 시에 아래와 같은 오류가 뜨면서 실행되지 않는경우
```
jupyter notebook --generate-config
vi ~/.jupyter/.jupyter_notebook_config.py
c.NotebookApp.browser = u'chrome'
```

