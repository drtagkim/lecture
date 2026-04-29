# 📊 범주형 데이터 분석 실습 가이드 (카이스퀘어 검정)

> **과목**: 빅데이터프로그래밍2  
> **주제**: 범주형 변수 간 독립성 검정 (Chi-Square Test of Independence)  
> **출제자**: 경희대학교 빅데이터응용학과 김태경 교수  

이 문서는 R을 활용하여 **범주형 변수(Categorical Variables)** 간의 관계를 분석하는 방법을 다룹니다. 카이스퀘어($\chi^2$) 검정을 통해 두 변수 간의 통계적 독립성을 확인하고, 이를 교차표(Cross Table)와 누적 막대 그래프(Stacked Bar Chart)로 시각화하는 전체 파이프라인을 실습합니다.

---

## 📁 사용 데이터셋 소개

실습에는 실제 캐글(Kaggle) 데이터셋 3종을 사용합니다. 아래 코드를 통해 모든 데이터를 불러올 수 있습니다.

| 번호 | 데이터셋 | 내용 및 출처 | 주요 변수 |
|:---:|---------|-------------|-------|
| 1 | **Titanic** | 1912년 타이타닉호 탑승객 생존 여부 기록 ([링크](https://www.kaggle.com/c/titanic)) | `Survived`, `Sex`, `Pclass` |
| 2 | **Mushroom** | 버섯의 물리적 특성과 독성 여부 분류 ([링크](https://www.kaggle.com/datasets/uciml/mushroom-classification)) | `class`, `odor`, `habitat`, `bruises` |
| 3 | **Adult Income** | 1994년 미국 센서스 데이터, 인구통계학적 요인과 연소득 5만 달러 초과 여부 ([링크](https://www.kaggle.com/datasets/uciml/adult-census-income)) | `sex`, `income`, `education`, `race` |

<br>

### 💻 0. 데이터 로드 (실습 전 준비사항)
각 문제의 데이터 파일을 R로 읽어오는 기초 작업입니다. `header = FALSE` 설정 및 `colnames` 지정을 통해 누락된 열 이름을 부여합니다.

```r
# 데이터가 있는 폴더를 워킹 디렉토리로 설정하거나 전체 경로를 지정하세요.

# 1. Titanic 데이터 로드 (헤더 포함)
titanic <- read.csv("titanic.csv")

# 2. Mushroom 데이터 로드 (헤더가 없으므로 직접 컬럼명 지정)
mushroom <- read.csv("mushrooms_raw.csv", header = FALSE)
colnames(mushroom) <- c("class","cap.shape","cap.surface","cap.color","bruises","odor",
                         "gill.attachment","gill.spacing","gill.size","gill.color",
                         "stalk.shape","stalk.root","stalk.surface.above","stalk.surface.below",
                         "stalk.color.above","stalk.color.below","veil.type","veil.color",
                         "ring.number","ring.type","spore.print.color","population","habitat")

# 3. Adult Census Income 데이터 로드 (문자열 앞뒤 공백 제거 위해 strip.white 적용)
adult <- read.csv("adult.csv", header = FALSE, strip.white = TRUE)
colnames(adult) <- c("age","workclass","fnlwgt","education","education.num",
                      "marital.status","occupation","relationship","race","sex",
                      "capital.gain","capital.loss","hours.per.week","native.country","income")
```

---

## 📝 1. 기초 실습: Titanic 탑승객 분석

첫 번째 단계에서는 가장 직관적인 타이타닉 생존율 분석을 통해 교차표 작성과 카이스퀘어 검정의 기초 문법을 익힙니다.

### 문제 1. 성별과 생존 여부 교차표 (Cross Table)
**목표**: `table()` 함수를 이용해 두 범주형 변수의 조합 빈도를 확인합니다.
* 힌트: `Survived`는 0/1 숫자이므로 `factor()`를 통해 "사망", "생존" 문자열로 변환하면 보기 편합니다.

**✅ 풀이 및 코드:**
```r
# 1) Survived 변수를 팩터형(Factor)으로 변환하여 범주형임을 명시
titanic$Survived_f <- factor(titanic$Survived, labels = c("사망", "생존"))

# 2) 교차 빈도표 생성: 행(Row)은 성별, 열(Column)은 생존 여부
t1 <- table(titanic$Sex, titanic$Survived_f)
print(t1)
```

**📊 해석 포인트:** 
결과를 보면 여성(female)은 생존자가 월등히 많고, 남성(male)은 사망자가 훨씬 많습니다. 이는 우연일까요, 아니면 성별과 생존 여부가 통계적으로 유의미한 관계를 맺고 있을까요? 다음 문제에서 이를 검증합니다.

<br>

### 문제 2. 성별과 생존 여부의 카이스퀘어 검정
**목표**: `chisq.test()` 함수로 교차표 데이터에 대한 독립성 검정을 수행합니다.

**✅ 풀이 및 코드:**
```r
# 앞서 만든 교차표 객체(t1)를 chisq.test 함수에 전달
test2 <- chisq.test(t1)
print(test2)
```

**📊 해석 심화 가이드:** 
* **가설 설정**:
  * 귀무가설(H₀): "타이타닉호에서 탑승객의 성별과 생존 여부는 아무런 관계가 없다(독립이다)."
  * 대립가설(H₁): "성별과 생존 여부는 관계가 있다."
* **결과 판독**: `p-value < 2.2e-16` (0.00000000000000022 미만)
* 유의수준 $\alpha = 0.05$보다 p-value가 월등히 작습니다. 따라서 귀무가설을 기각합니다. **성별과 생존 여부는 강력한 상관관계가 있음**을 통계적으로 입증했습니다.

<br>

### 문제 3. 좌석 등급(Pclass)과 생존 시각화
**목표**: 좌석 등급과 생존 여부의 관계를 검정하고, 이를 100% 누적 막대 그래프(`ggplot2`)로 시각화합니다.

**✅ 풀이 및 코드:**
```r
library(ggplot2)

# 1) 좌석 등급 레이블링
titanic$Pclass_f <- factor(titanic$Pclass, labels = c("1등석", "2등석", "3등석"))

# 2) 교차표 및 카이스퀘어 검정
t3 <- table(titanic$Pclass_f, titanic$Survived_f)
test3 <- chisq.test(t3)
print(test3)  # p-value < 2.2e-16 (귀무가설 기각)

# 3) 100% 누적 막대 그래프 생성 (position = "fill"이 핵심)
ggplot(titanic, aes(x = Pclass_f, fill = Survived_f)) +
  geom_bar(position = "fill", alpha = 0.8) +
  scale_y_continuous(labels = scales::percent) +
  theme_minimal() +
  labs(x = "좌석 등급", y = "생존 비율 (%)", fill = "상태",
       title = "타이타닉 좌석 등급별 생존 비율")
```

**📊 실무 시사점:** 
1등석의 생존율이 3등석에 비해 확연히 높습니다. p-value가 0.05보다 작으므로 "재난 상황에서 좌석 등급(재력/사회적 지위)은 생존에 영향을 미쳤다"는 가설이 성립합니다.

---

## 📝 2. 응용 실습: 독버섯(Mushroom) 판별 분석

버섯의 물리적 특성만으로 독버섯 여부를 감별할 수 있는지 통계적으로 확인합니다. 이 데이터는 범주형 변수의 차원이 큰 경우(냄새 종류 9개 등)의 카이스퀘어 검정을 다룹니다.

### 문제 4 & 5. 냄새(odor) 및 멍(bruises)과 독성(class)의 관계
**목표**: 여러 카테고리를 가진 변수에 대한 검정을 수행합니다.

**✅ 풀이 및 코드:**
```r
# class 레이블링 (e: 식용, p: 독버섯)
mushroom$class_label <- ifelse(mushroom$class == "e", "edible", "poisonous")

# odor(냄새) 레이블링
mushroom$odor_label <- factor(mushroom$odor,
  levels = c("a","c","f","l","m","n","p","s","y"),
  labels = c("almond","creosote","foul","anise","musty","none","pungent","spicy","fishy"))

# 1. 냄새와 독성의 관계 확인
t4 <- table(mushroom$odor_label, mushroom$class_label)
print(t4)
# 주의: 특정 냄새(예: almond)는 독버섯 빈도가 0입니다. 이 경우 카이스퀘어 검정 시 '기대빈도' 경고가 뜰 수 있습니다.

# 2. 멍(bruises)과 독성의 카이스퀘어 검정
mushroom$bruises_label <- ifelse(mushroom$bruises == "t", "yes", "no")
t5 <- table(mushroom$bruises_label, mushroom$class_label)

test5 <- chisq.test(t5)
print(test5)
```

**📊 분석 인사이트:** 
교차표(t4)를 확인하면, **냄새가 almond(아몬드), anise(아니스)인 버섯은 100% 식용**이었으며, **foul(썩은내), pungent(자극적인 냄새) 등은 100% 독버섯**이었습니다. 범주형 변수 중 특정 카테고리가 예측률 100%를 보일 때, 이는 머신러닝 의사결정나무(Decision Tree) 등에서 분기점으로 사용하기 매우 좋은 특성(Feature)이 됩니다.

<br>

### 문제 6. 서식지(habitat)와 독성(class)
**목표**: 장소에 따라 독버섯 발견 확률이 다른지 검정합니다.

**✅ 풀이 및 코드:**
```r
mushroom$habitat_label <- factor(mushroom$habitat,
  levels = c("d","g","l","m","p","u","w"),
  labels = c("woods","grasses","leaves","meadows","paths","urban","waste"))

t6 <- table(mushroom$habitat_label, mushroom$class_label)
test6 <- chisq.test(t6)
print(test6)
```

---

## 📝 3. 심화 실습: Adult Income 불평등 요인 분석

1994년 미국 인구조사 데이터를 바탕으로, 소득(연 5만 달러 초과 여부)과 다양한 인구통계학적 특성 간의 상호작용을 분석합니다.

### 문제 7 & 8. 성별(sex)과 소득 불평등 분석
**✅ 풀이 및 코드:**
```r
# 성별과 소득의 교차표 생성
t7 <- table(adult$sex, adult$income)
print(t7)

# 독립성 검정
test8 <- chisq.test(t7)
print(test8)
```
**📊 사회학적 해석:** 
카이스퀘어 값이 $1517.8$로 매우 높으며, $p < 0.05$입니다. 이는 통계적으로 우연히 발생할 수 없는 강력한 소득 격차가 성별 간에 존재함을 시사합니다. (유리천장 효과의 정량적 증거)

<br>

### 문제 9. 연속형 변수(교육 연수)의 범주화 및 교차 검정
**목표**: `education.num`과 같은 연속/이산 수치형 데이터를 임의의 그룹으로 묶어(Binning) 범주형 데이터로 변환한 뒤 검정합니다.

**✅ 풀이 및 코드:**
```r
# ifelse를 중첩하여 Low(8년 이하), Mid(9~12년), High(13년 이상)으로 범주화
adult$edu_group <- ifelse(adult$education.num <= 8, "Low",
                   ifelse(adult$education.num <= 12, "Mid", "High"))

t9 <- table(adult$edu_group, adult$income)
test9 <- chisq.test(t9)
print(test9)
```

<br>

### 문제 10. 인종(race)별 소득 수준 시각화 및 검정 종합
**목표**: 이번 실습의 최종 단계로, 카이스퀘어 검정과 논문/보고서용 고품질 시각화를 종합합니다.

**✅ 풀이 및 코드:**
```r
# 교차표 및 검정
t10 <- table(adult$race, adult$income)
test10 <- chisq.test(t10)
print(test10)

# 시각화 (인종별 소득 비율 누적 막대 그래프)
ggplot(adult, aes(x = race, fill = income)) +
  geom_bar(position = "fill", alpha = 0.8, color = "white") +
  scale_y_continuous(labels = scales::percent) +
  scale_fill_brewer(palette = "Set1") + # 색상 팔레트 변경
  theme_minimal(base_size = 14) +
  theme(axis.text.x = element_text(angle = 45, hjust = 1)) +
  labs(x = "Race (인종)", y = "Proportion (비율)", fill = "Income",
       title = "미국 인종별 소득 비율 불평등 (1994 Census)")
```

---

## 📌 핵심 요약 (Takeaway)

### R 함수 치트시트
* **`table(Var1, Var2)`**: 두 범주형 변수의 발생 빈도 집계 (교차표 작성)
* **`chisq.test(Table)`**: 카이스퀘어 통계량 산출. $H_0$는 "두 변수는 서로 관계가 없다(독립이다)". p-value가 0.05보다 작으면 $H_0$ 기각.
* **`factor(Variable, labels=c(...))`**: 숫자로 코딩된 범주형 데이터를 사람이 읽을 수 있는 문자 레이블로 맵핑.

### ⚠️ 카이스퀘어 검정 시 주의사항
교차표의 각 셀에 대한 **기대 빈도(Expected Frequency)가 5 미만인 셀이 전체의 20% 이상**일 경우 카이스퀘어 통계량의 신뢰도가 떨어집니다. 이 경우 R 시스템이 자동 경고 메시지를 띄우며, 대안으로 **피셔의 정확 검정(`fisher.test()`)**을 사용해야 합니다.
