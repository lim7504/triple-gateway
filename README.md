# 과제. 트리플여행자 클럽 마일리지 서비스

### 프로젝트 구성

1. triple-gateway : [https://github.com/lim7504/triple-gateway](https://github.com/lim7504/triple-gateway)
2. triple-eureka : [https://github.com/lim7504/triple-eureka](https://github.com/lim7504/triple-eureka)
3. triple-event : [https://github.com/lim7504/triple-event](https://github.com/lim7504/triple-event)
4. triple-review : [https://github.com/lim7504/triple-review](https://github.com/lim7504/triple-review)
5. triple-point : [https://github.com/lim7504/triple-point](https://github.com/lim7504/triple-point)

### 서비스 구동방법

1. 위 프로젝트 구성에 안내된 모든 프로젝트를 내려받아 실행합니다.
2. 유레카 서버에 각 서비스들이 등록되었는지 확인합니다.
    1. 브라우져에서 [http://localhost:8761/](http://localhost:8761/) 경로로 접근합니다.
    2. [Instances currently registered with Eureka] 부분에
        1. APIGATEWAY-SERVICE 
        2. EVENT-SERVICE
        3. POINT-SERVICE
        4. REVIEW-SERVICE
        
        4개의 서비스가 등록 되었는지 확인합니다.
        
3. 아래 Mysql DataBase 스키마를 실행합니다.

```sql
create database triple;
use triple;

create table event_history
(
	id varchar(36) null,
	type varchar(20) not null,
	action varchar(10) not null,
	user_id varchar(36) not null,
	json text not null,
	created_date timestamp default CURRENT_TIMESTAMP not null,
	modified_date timestamp default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP
);

create table place
(
	id varchar(36) not null
		primary key,
	name varchar(20) not null,
	created_date timestamp default CURRENT_TIMESTAMP not null,
	modified_date timestamp default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP
);

INSERT INTO place (id, name) VALUES ('603178bc-214a-49f0-85ea-f1cacbeeaa41', '서울');
INSERT INTO place (id, name) VALUES ('603178bc-214a-49f0-85ea-f1cacbeeaa45', '제주도');

create table user
(
	id varchar(36) not null
		primary key,
	name varchar(20) not null,
	point int default 0 not null,
	created_date timestamp default CURRENT_TIMESTAMP not null,
	modified_date timestamp default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP
);

INSERT INTO user (id, name, point) VALUES ('04ea2107-7b9e-4360-8099-6f10ce2276ac', '홍길동', 0);
INSERT INTO user (id, name, point) VALUES ('04ea2107-7b9e-4360-8099-6f10ce2276ad', '임꺽정', 0);
INSERT INTO user (id, name, point) VALUES ('04ea2107-7b9e-4360-8099-6f10ce2276af', '둘리', 0);

create table review
(
	id varchar(36) not null
		primary key,
	place_id varchar(36) null,
	user_id varchar(36) null,
	content varchar(1000) not null,
	del_yn bit not null,
	first_review_yn bit not null,
	created_date timestamp default CURRENT_TIMESTAMP not null,
	modified_date timestamp default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP,
	constraint review_place_id_fk
		foreign key (place_id) references place (id),
	constraint review_user_id_fk
		foreign key (user_id) references user (id)
);

create index review_place_id_index
	on review (place_id);

create index review_user_id_place_id_index
	on review (user_id, place_id);

create table review_photo
(
	id varchar(36) not null
		primary key,
	review_id varchar(36) null,
	image_id varchar(36) not null,
	del_yn bit not null,
	created_date timestamp default CURRENT_TIMESTAMP not null,
	constraint photo_review_id_fk
		foreign key (review_id) references review (id)
);

create table user_point_history
(
	id varchar(36) not null
		primary key,
	user_id varchar(36) null,
	type varchar(10) not null,
	reason varchar(1000) not null,
	apply_point int null,
	accumulated_point int not null,
	created_date timestamp default CURRENT_TIMESTAMP not null,
	modified_date timestamp default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP,
	constraint user_point_history_user_id_fk
		foreign key (user_id) references user (id)
);
```

### 기준 데이터

1. User 정보
    1. id : 04ea2107-7b9e-4360-8099-6f10ce2276ac
    name : 홍길동
    2. id : 04ea2107-7b9e-4360-8099-6f10ce2276ad
    name : 임꺽정
    3. id : 04ea2107-7b9e-4360-8099-6f10ce2276af
    name : 둘리
2. Place 정보
    1. id : 603178bc-214a-49f0-85ea-f1cacbeeaa41
    name : 서울
    2. id : 603178bc-214a-49f0-85ea-f1cacbeeaa45
    name : 제주도

### API 리스트

1. 리뷰 추가/수정/삭제 API
    1. 리뷰 추가
        1. POST [http://localhost:8000/event-service/events](http://localhost:8000/event-service/events)
        2. ReuqestBody
            
            ```json
            {
                "type" : "REVIEW",
                "action" : "ADD",
                "userId" : "04ea2107-7b9e-4360-8099-6f10ce2276ad",
                "content" : "좋아요!",
            		"attachedPhotoIds" : ["qeafa503-7d28-4cdd-bfe5-d8aba810f9f6","qb5f4de6-1714-4973-8d4f-c9a68e1a1a6e"],
                "placeId" : "603178bc-214a-49f0-85ea-f1cacbeeaa41"
            }
            ```
            
        3. ResponseBody
            
            ```json
            {
                "code": "CM_0001",
                "data": {
                    "reviewId": "10c1d51f-a886-4c37-9589-57f7296fc55f"
                },
                "timeStamp": "2021-12-29T20:55:53.764743"
            }
            ```
            
    2. 리뷰 수정
        1. POST [http://localhost:8000/event-service/events](http://localhost:8000/event-service/events)
        2. ReuqestBody
            
            ```json
            {
                "type" : "REVIEW",
                "action" : "MOD",
                "userId" : "04ea2107-7b9e-4360-8099-6f10ce2276ad",
                "reviewId" : "10c1d51f-a886-4c37-9589-57f7296fc55f",
                "attachedPhotoIds" : ["qeafa503-7d28-4cdd-bfe5-d8aba810f9f6","qb5f4de6-1714-4973-8d4f-c9a68e1a1a6e"],
                "content" : "좋아요!!!!!!"
            }
            ```
            
        3. ResponseBody
            
            ```json
            {
                "code": "CM_0001",
                "timeStamp": "2021-12-29T21:41:39.261083"
            }
            ```
            
    3. 리뷰 삭제
        1. POST [http://localhost:8000/event-service/events](http://localhost:8000/event-service/events)
        2. ReuqestBody
            
            ```json
            {
                "type" : "REVIEW",
                "action" : "DELETE",
                "userId" : "04ea2107-7b9e-4360-8099-6f10ce2276ad",
                "reviewId" : "10c1d51f-a886-4c37-9589-57f7296fc55f"
            }
            ```
            
        3. ResponseBody
            
            ```json
            {
                "code": "CM_0001",
                "timeStamp": "2021-12-29T21:41:39.261083"
            }
            ```
            
2. 포인트 조회
    1. GET [http://localhost:8000/point-service/point/users/04ea2107-7b9e-4360-8099-6f10ce2276ad](http://localhost:8000/point-service/point/users/04ea2107-7b9e-4360-8099-6f10ce2276ad)
    2. ResponseBody
        
        ```json
        {
            "code": "CM_0001",
            "data": {
                "userId": "04ea2107-7b9e-4360-8099-6f10ce2276ad",
                "accumulatedPoint": 3,
                "logs": [
                    "type=ADD, applyPoint=1, accumulatedPoint=3, reason='review point : reviewId = '8844ba11-50ac-448d-8b9d-88e8b9596871', photo = 1', created_date='2021-12-29T21:56:16+09:00",
                    "type=DELETE, applyPoint=1, accumulatedPoint=2, reason='review point : reviewId = '8844ba11-50ac-448d-8b9d-88e8b9596871', photo = 1', created_date='2021-12-29T21:55:35+09:00",
                    "type=ADD, applyPoint=3, accumulatedPoint=3, reason='review point : reviewId = '8844ba11-50ac-448d-8b9d-88e8b9596871', review = 1, photo = 1, firstReview = 1', created_date='2021-12-29T21:54:48+09:00"
                ]
            },
            "timeStamp": "2021-12-29T21:58:16.794054"
        }
        ```
