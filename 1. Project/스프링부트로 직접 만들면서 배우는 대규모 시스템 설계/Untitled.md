
**todo.**
- service에 dto 클래스 보면 response에 AllArg* 있는데 굳이 필요하지 않아 보인다. 
	- 오히려 pojo 방식으로 선언해서 할당하는게 가독성이 좋아 보임
	- article, comment 모듈 수정하기 


## 게시글 

<img src="./image/article_erd.png"/>




## 댓글 

> **대댓글 (2depth) 방식**과 **무한 depth 방식** 둘 다 구현

<img src="./image/comment_erd.png"/>
- **대댓글** : comment_parent_id 기반으로 계층형 구조
- **무한 depth**: varchar(25) path, 25개 문자열 기반으로 5개씩 나눠 계층 표현
	- 총 5depth 존재 00000 00000 00000 00000 ~ zzzzz zzzzz zzzzz zzzzz zzzzz
	- 각 자리는 **0-9A-Za-z**, 총 **62개** 문자열 중 하나
		- 텍스트 선언 후 인덱스 기반으로 다음 문자열 가져와 처리
	- depth 별로 댓글은 62^5 = 약 9억개 정도 할당 가능 
	- depth 별로 댓글이 9억개가 넘어가면 문자열 길이(path)를 늘리거나 하면 됨
- **삭제**의 경우 
	- 자식이 있으면 논리적 삭제(`deleted = true`업데이트)
	- 자식이 없으면 물리적 삭제(DB)

**두 방식 비교.**



**TODO.**
- 게시판을 두개다 만들건지 아니면 하나만 할건지 선택해서 남겨두자 


## 좋아요

> **비관적 락(pessmistic lock)**, **낙관적 락(optimistick lock)** 방식을 둘 다 사용해 봄

| 기능     | REST 표현                | HTTP 메서드 | 설명                   |
| ------ | ---------------------- | -------- | -------------------- |
| 좋아요    | `/articles/{id}/likes` | `POST`   | 좋아요 추가 (Like 리소스 생성) |
| 좋아요 해제 | `/articles/{id}/likes` | `DELETE` | 좋아요 취소 (Like 리소스 제거) |
- 좋아요 응답은 201 created
- 좋아요 해제 응답은 204 no content

|서비스 유형|추천 방식|
|---|---|
|**정합성 최우선 (재고/주문)**|비관적 락 (Pessimistic Lock)|
|**많은 트래픽, 정확성 중요 (좋아요)**|낙관적 락 + 재시도, 또는 Redis 누적 후 DB 반영|
|**캐시 허용 가능 (조회수, 좋아요)**|Redis 기반 처리 (성능 우선)|
|**낮은 충돌 가능성 (로그 등)**|락 없이 처리|
- 비관적 락을 사용하는 경우 
	- update 방식과 select .. for update 방식이 있음
	- 정합성은 높으나 데드락 위험 있음
- 낙관적 락을 사용하는 경우 
	- 재시도 로직이 필요함 (spring-retry)
	- 재시도 로직이 없는 경우 데이터 유실 높음


> 통합 테스트만 작성하는 걸로 함, 왜냐하면 service, repository 둘다 의미 없어 보이고 
> 동시성 이슈에 대한 확인만 하면 되기 때문에 controller 단에서만 테스트하는게 맞다고 생각
> - 처음에는 강의와 같이 비관적 락 (2개, `update`, `select .. for update`), 낙관적 락 테스트(재시도 x)
> - `spring-retry` 추가해서 낙관적 락만 사용하는 걸로 최종 결정 후 리팩터링 


낙관적락 테스트시 beforeEach에서 데이터 초기화하면 바로 OptimisticLockingFailureException 발생  > beforeEach를 빼면 정상 실행은 되나 재시도 로직이 없어 전체 1000개 실패 

retry를 포함할 경우 41초 걸리나 1000개 모두 성공 

1000개 넣는데 
비관적 락1(update) 300ms
비관적 락2(select for update) 1156ms
낙관적 락 (재시도x) 테스트 실패..




**TODO.**
- `userId`를 컨트롤러에서 path로 받는데 스프링 시큐리티나 그런걸로 주입받아 사용하는게 좋을듯함 
	- 정책 : 로그인한 사용자에 한해서 `좋아요/좋아요 해제` 가능


## 조회수

