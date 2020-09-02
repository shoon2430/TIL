# Django - Model - Field

오랜만에 장고로 토이프로젝트를 진행하는데 전부터 정리를 해둘걸 후회가 된다.

 프로젝트를 진행하면서 찾아보거나 필요하다고 싶은 내용을 개인적으로 정리해나가려고 한다.



## model

장고프로젝트를 처음 시작하면 모델부터 구성하게되는것같다. 



### [Django 모델필드 옵션 및 타입](https://docs.djangoproject.com/en/3.1/ref/models/fields/)

#### Field types

 `IntegerField` `CharField` `DateField` `FileField` `ImageField` `TextField` `BooleanField`

#### Relationship fields

`ForeignKey` `ManyToManyField` `OneToOneField`



#### Field Option

- null (DB 옵션) : DB 필드에 NULL 허용 여부 (디폴트 : False)
- unique (DB 옵션) : 유일성 여부 (디폴트 : False)
- blank : 입력값 유효성 (validation) 검사 시에 empty 값 허용 여부 (디폴트 : False)
- default : default 값 지정
- verbos_name : 필드 레이블, 지정되지 않으면 필드명이 쓰여짐
- validateors : 입력값 유효성 검사를 수행할 **함수** 를 다수 지정 
  - ex) 평점( 1 ~ 5 )점 만 혀용
- choises (form widget 용): select box 소스로 사용
- help_text (form widget 용) : 필드 입력 도움말
- auto_now_add : Bool, True 인 경우, 레코드 생성시 현재 시간으로 자동 저장 



#### 예시

```python
def rating_validator(value):
    if value<0 and value>5:
        raise ValidationError('Invalid LngLat Type')

class Item(models.Model):
 
    choicesItem = (
              ('제목1', '제목 1 레이블'),
              ('제목2', '제목 2 레이블'),
              ('제목3', '제목 3 레이블'),
    )

    title = models.CharField(max_length=100, help_text='최대 100자 내로 입력가능합니다.', choices = choicesItem)
    content = models.TextField(verbose_name='내용')
    tags = models.CharField(max_length=100, blank=True)
    rating = models.CharField(max_length=50, blank=True, 
                              validators = [rating_validator], # 함수를 넘겨서 유효성 검사 실행 
                              help_text='1 ~ 5 값 입력')
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
```

