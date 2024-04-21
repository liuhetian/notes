# Langchain

## 读取和加载文档

```
from langchain_community.document_loaders.recursive_url_loader import RecursiveUrlLoader
url = "https://docs.python.org/3.9/"
loader = RecursiveUrlLoader(url=url, max_depth=2)
docs = loader.load()
```   
