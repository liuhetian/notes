# Langchain

## 读取和加载文档

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import  PromptTemplate, format_document
from langchain_core.output_parsers import JsonOutputParser, StrOutputParser
from langchain_community.document_loaders import ArxivLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter

model = ChatOpenAI(
    base_url='http://api.liuhetian.work:3000/v1',
    api_key='<api_key>', 
    model='gpt-3.5-turbo-0125',
    temperature=0,
)
```

核心部分：
```python
# langchain中有很多的文档加载器，langchain_community.document_loaders里面还有TextLoader等
loader = ArxivLoader(query='2210.03629', load_max_docs=1)  
docs = loader.load()  # 好像还可以懒加载

# chunks[0].metadata, chunks[0].page_content
```

```python
# 配置如何划分
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=0)
# 划分之后，每个小chunk类型好像还是一样的，都还是有metadata和page_content属性
chunks = text_splitter.split_documents(docs)
```

最后开始工作，要注意
```python
# 下面这个函数就是把文档块的内容放进模版里
format_document(chunks[0], PromptTemplate.from_template("{page_content}"))
```

```python
# 但是没想明白下面这么绕了一大圈，不还是直接把内容放进去
doc_prompt = PromptTemplate.from_template("{page_content}")
chain = (
    {
        "content": lambda docs: '\n\n'.join(format_document(doc, doc_prompt) for doc in docs)
    }
    | PromptTemplate.from_template("使用中文总结以下内容，不需要人物介绍，字数控制在50个字符以内：\n\n{content}")
    | model
    | StrOutputParser()
)
chain.invoke(chunks[:5])
```
