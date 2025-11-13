
> 웹 콘솔 지원하지 않는다. 
>Azure Sotrage Explorer 사용해서 연결 

**설치**

```shell
docker run -d -p 10000:10000 -p 10001:10001 -p 10002:10002 --name azurite-local mcr.microsoft.com/azure-storage/azurite
```

```text
// local.settings.json 의 Values에 아래 항목 추가하면 로컬에 설치된 Azurite를 찾는다.. 그러다보니 Azurite 꺼져 있으면 .. 실패함
"StorageQueue": "UseDevelopmentStorage=true"
```


**Azure Storage Explorer**
- [Azure Storage Explorer – 클라우드 저장소 관리 | Microsoft Azure](https://azure.microsoft.com/ko-kr/products/storage/storage-explorer/?msockid=3524f9483223634a38d5ef123336626b)
- [Azurite를 사용하여 자동화된 테스트 실행 - Azure Storage | Microsoft Learn](https://learn.microsoft.com/ko-kr/azure/storage/blobs/use-azurite-to-run-automated-tests?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&bc=%2Fazure%2Fstorage%2Fblobs%2Fbreadcrumb%2Ftoc.json)

표시 이름 : `local-1`
계정 이름 : `devstoreaccount1`
계정키 : `Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==`
Blob 포트 : 10000
큐 포트 : 10001
테이블 포트 : 10002