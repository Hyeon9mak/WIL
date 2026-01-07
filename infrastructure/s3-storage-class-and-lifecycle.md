## 🪣 S3 Storage Class

S3 Storage Class 는 AWS S3 에 저장된 객체(파일) 들의 저장 유형을 정의하는 설정이다.
모든 Class 는 기본적으로 내구성, 가용성, 가용성 SLA 들을 기본적으로 99% 이상 제공한다.
모든 Storage Class 들은 내구성과 가용성은 높게 보장되니, 사용자는 **데이터 종류**, **접근 빈도** 2가지 요소에 따라 적절한 Storage Class 를 선택하면 된다.
즉, 사용 패턴을 명확히 진단하면 쓴 만큼만 비용을 지불하는 효율적인 Storage 운영이 가능하다.

더 자세한 비교는 아래 표를 참고하자.

| Storage Class                     | 내구성 (Durability)       | 가용성 (Availability) | SLA   | 가용 영역 (AZ) | 객체당 최소 용량 요금 | 최소 Storage 기간 요금 | 검색 요금  | 검색 지연 시간  |
|-----------------------------------|------------------------|--------------------|-------|------------|--------------|------------------|--------|-----------|
| **S3 Standard**                   | 99.999999999% (11 9's) | 99.99%             | 99.9% | ≥3         | -            | -                | -      | 밀리초       |
| **S3 Intelligent-Tiering**        | 99.999999999% (11 9's) | 99.9%              | 99%   | ≥3         | 128KB        | -                | -      | 밀리초       |
| **S3 Express One Zone**           | 99.999999999% (11 9's) | 99.95%             | 99.9% | 1          | -            | 1시간              | GB당 요금 | 한 자릿수 밀리초 |
| **S3 Standard-IA**                | 99.999999999% (11 9's) | 99.9%              | 99%   | ≥3         | 128KB        | 30일              | GB당 요금 | 밀리초       |
| **S3 One Zone-IA**                | 99.999999999% (11 9's) | 99.5%              | 99%   | 1          | 128KB        | 30일              | GB당 요금 | 밀리초       |
| **S3 Glacier Instant Retrieval**  | 99.999999999% (11 9's) | 99.9%              | 99%   | ≥3         | 128KB        | 90일              | GB당 요금 | 밀리초       |
| **S3 Glacier Flexible Retrieval** | 99.999999999% (11 9's) | 99.99%             | 99.9% | ≥3         | 40KB         | 90일              | GB당 요금 | 분~12시간    |
| **S3 Glacier Deep Archive**       | 99.999999999% (11 9's) | 99.99%             | 99.9% | ≥3         | 40KB         | 180일             | GB당 요금 | 9~48시간    |

- S3 Intelligent-Tiering 은 128KB 미만 객체도 저장 가능하나, 항상 Frequent Access tier 요금으로 과금되며 모니터링 요금은 부과되지 않음
- S3 One Zone-IA는 단일 AZ에 저장되므로 AZ 손실 시 데이터 손실 가능
- S3 Glacier Flexible Retrieval 과 Deep Archive 각 객체당 추가 메타데이터 40KB(S3 Standard 8KB + Glacier 32KB) 요금 발생

#### S3 Standard

- 이름 그대로 가장 일반적인 Storage Class
- 높은 내구성과 가용성을 제공하며, 지연 시간이 짧아 빠른 데이터 접근 가능
- 자주 접근하는 데이터를 위함

#### S3 Intelligent-Tiering

- 3개의 Access tier (Frequent, Infrequent, Archive) 로 구성
- 접근 패턴에 따라 자동으로 Access tier 간 전환
- 접근 빈도가 예측 불가능한 데이터를 위한 Storage Class

#### S3 Express One Zone

- S3 Standard 보다 최대 10배 빠른 데이터 접근 속도 제공
- 자주 Access 하는 데이터와 까다로운 애플리케이션 설계에 적합
- 단일 가용 영역(AZ)에 저장되어 비용 절감
- 자주 접근하는 데이터에 적합하지만, AZ 손실 시 데이터 손실

#### S3 Standard-IA

- 자주 접근하지 않는 데이터를 위한 Storage Class
- 높은 내구성과 가용성을 제공하며, 지연 시간은 S3 Standard 와 유사
- 접근 빈도가 낮은 데이터를 위한 비용 효율적인 옵션

#### S3 One Zone-IA

- 단일 가용 영역(AZ)에 저장되어 비용 절감
- 접근 빈도가 낮은 데이터를 위한 Storage Class
- AZ 손실 시 데이터 손실 가능
- S3 Standard-IA 와 유사한 요금 구조

### S3 Glacier

Glacier(빙하) 라는 이름에서 알 수 있듯, 장기 보관을 위한 Storage Class 다.
데이터 접근 빈도가 매우 낮고, 장기간 보관이 필요한 데이터를 위한 옵션이다.

다른 Storage Class 와 달리, Glacier 는 완전히 별개의 저장소에 데이터를 보관한다.
때문에 S3 에서 제공하는 다른 기능들을 이용할 수 없으며, S3 Service 에서 Access 가 불가능하다.
(복원 작업을 통해서만 접근 가능. 분~시간 단위가 소요된다.)
즉 **마음대로 검색이 불가능하며, 마음대로 삭제 또한 불가능**하다.
Glacier 는 장기간 보관을 주 목적으로 설계한 서비스이고, 그 최소한의 조건이 3개월이기 때문이다.
만약 3개월 이전에 삭제할 경우, 잔여 기간에 대한 요금이 청구된다.

대신 그만큼 저장 비용이 매우 저렴하므로, 보관 특성에 따라 활용을 결정하면 된다.

#### S3 Glacier Instant Retrieval

- Glacier Storage Class 중 유일하게 밀리초 단위의 지연 시간 제공
- 자주 접근하지는 않지만, 필요 시 즉시 접근해야 하는 데이터를 위한 옵션
- 분기 1회 Access 에 적합

#### S3 Glacier Flexible Retrieval

- 분~시간(최대 12시간) 단위의 지연 시간 제공
- 연 1회 Access 에 적합
- 장기 보관이 필요하며, 접근 빈도가 매우 낮은 데이터를 위한 옵션

#### S3 Glacier Deep Archive

- 가장 저렴한 저장 비용 제공
- 9~48시간 단위의 지연 시간 제공
- 연 1회 미만 Access 에 적합
- 금융, 의료 등 데이터를 초장기간 보관하는 서비스와 고객을 위해 설계

<br>

## 🪣 S3 Storage Class 변경하기

S3 bucket console 에서 대상 객체를 선택한 후, `Actions` - `Change storage class` 를 선택하면 쉽게 변경할 수 있다.

<img width="1258" height="645" alt="Image" src="https://github.com/user-attachments/assets/22931a1a-8dda-473b-b675-27a15777b792" />

보통 S3 에 적게는 수백, 수천에서 많게는 수억, 수십억 개의 객체가 저장하곤 한다.
개발자가 하나하나 수동으로 변경하는 것은 지나치게 비효율적이고, 사실상 불가능에 가깝다.
그리고 S3 는 이러한 문제를 보완하기 위해 Lifecycle(생명주기) 개념을 제공한다.

<br>

## 🪣 S3 Lifecycle

S3 bucket 에 저장된 객체의 생명주기(Lifecycle)을 관리하는 개념이다.
지정한 기간이 경과하면, 객체를 자동으로 다른 Storage Class 로 전환하거나 삭제할 수 있다.
즉, 자신의 서비스 특성에 맞게 S3 객체의 저장 정책을 자동화할 수 있다.
S3 Lifecycle 은 크게 2가지 종류가 있다.

### Storage Class Transition

- 지정한 기간이 경과하면, 객체를 자동으로 다른 Storage Class 로 전환
- 예: 30일 후 S3 Standard-IA 로 전환, 90일 후 S3 Glacier Flexible Retrieval 로 전환 등

단, 전환 대상 Storage Class 에 따라 최소 보관 기간이 존재한다.
예를 들어, S3 Standard-IA 로 전환한 객체는 최소 30일 동안 보관해야 하며,
S3 Glacier Flexible Retrieval 로 전환한 객체는 최소 90일 동안 보관해야 한다.
따라서 전환 시점과 최소 보관 기간을 고려하여 정책을 설정해야 한다.

또한 Lifecycle 을 통해 Storage Class 를 전환할 땐 Waterfall 형태의 전환만 가능하다.
(역방향 전환 불가능) 아래 그림을 참고하자.

<img src="https://docs.aws.amazon.com/images/AmazonS3/latest/userguide/images/lifecycle-transitions-v4.png">

### Expiration

- 지정한 기간이 경과하면, 객체를 자동으로 삭제
- 예: 365일 후 객체 삭제

2가지 종류를 조합하여, 장기 보관이 필요한 데이터를 저비용으로 관리할 수 있다.
예를 들어, 자주 접근하지 않는 로그 데이터를 S3 Standard-IA 로 전환한 후, 1년 후 삭제하는 정책을 설정할 수 있다.

<br>

## 🪣 S3 Lifecycle rule 설정하기

S3 bucket console 에서 `Management` - `Lifecycle rules` - `Create lifecycle rule` 을 선택하여 설정할 수 있다.

<img width="1252" height="545" alt="Image" src="https://github.com/user-attachments/assets/2f94ce8c-4cb9-4d61-a0ae-616397efd74d" />

`Limit the scope of this rule using one or more filters` 옵션을 통해 특정 prefix, tag 혹은 객체의 크기에 따라 규칙을 적용 대상을 한정할 수 있다.
`Apply to all objects in the bucket` 옵션을 선택하면, 버킷 내 모든 객체에 규칙이 적용된다.

<img width="1239" height="708" alt="Image" src="https://github.com/user-attachments/assets/64aa866e-5b2d-4ca6-b925-ee59ce54b3c5" />

`Lifecycle rule actions` 항목에선 5가지 작업을 설정할 수 있는데, 앞서 알아본 2가지 개념으로 나누어 볼 수 있다.

- `Transition current versions of objects between storage classes`
    - Storage Class Transition
    - 경과 기간 선택 후, 전환할 Storage Class 선택
- `Transition noncurrent versions of objects between storage classes`
    - Storage Class Transition
    - 버전 관리가 활성화된 버킷에서, 이전 버전 객체에 대해 Storage Class 전환 선택
- `Expire current versions of objects`
    - Expiration
    - 현재 버전 객체에 대해 만료 처리, 추가 기간 경과 후 영구 삭제
- `Permanently delete noncurrent versions of objects`
    - Expiration
    - 객체의 이전 버전 영구 삭제
- `Delete expired object delete markers or incomplete multipart uploads`
    - Expiration
    - 현재 날짜 기준으로 삭제 실행

<img width="1225" height="299" alt="Image" src="https://github.com/user-attachments/assets/89a98102-0b0e-412c-8b90-6d09ca4bdf37" />

각 사용 패턴에 맞게 선정 후 최종 생성을 마치면, 설정이 완료된다.

AWS S3 는 Lifecycle rule 적용 기준을 익일 00:00 UTC 로 정하고 있다.
예를 들어 `2026-01-08 00:05 UTC` 에 생성한 객체에 대해 "10일 후 S3 Standard-IA 로 전환" 규칙을 적용하면, `2026-01-19 00:00 UTC` 에 전환이 실행된다.
이 점 유의하여 정책을 설계하면 좋겠다.

<br>

## References

- [AWS S3 Storage Classes 공식 페이지](https://aws.amazon.com/s3/storage-classes/)
- [AWS S3 Pricing 공식 페이지](https://aws.amazon.com/s3/pricing/)
- [AWS S3 User Guide - Storage Classes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)
- [Amazon S3 Express One Zone 고성능 스토리지 클래스 정식 출시](https://aws.amazon.com/ko/blogs/korea/new-amazon-s3-express-one-zone-high-performance-storage-class/)
- [S3 Intelligent-Tiering 작동 방식](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/userguide/intelligent-tiering-overview.html)
- [Amazon S3 Glacier Instant Retrieval 신규 스토리지 클래스 출시](https://aws.amazon.com/ko/blogs/korea/amazon-s3-glacier-is-the-best-place-to-archive-your-data-introducing-the-s3-glacier-instant-retrieval-storage-class/)
- [Transitioning objects using Amazon S3 Lifecycle](https://docs.aws.amazon.com/AmazonS3/latest/userguide/lifecycle-transition-general-considerations.html)
