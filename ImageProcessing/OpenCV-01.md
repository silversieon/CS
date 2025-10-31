# 1장

---

## 영상 처리

**영상처리(image processing)는 입력된 영상을 어떤 목적을 위해 처리하는 기술**

영상을 입력 받아서 어떤 목적을 위해 처리해 새로운 것을 얻는데, 이 과정에서 처리 결과인 출력이

“영상 자체”인 경우와 “영상의 특성”인 경우로 나눌 수 있음

- 영상 자체 : 저 수준의 영상처리
    - 잡음 제거, 영상 향상 처리 (화소에 대한 처리)
- 영상의 특성들 : 고 수준의 영상처리
    - 특징추출, 영상표현, 물체인식 등

## 컴퓨터 비전(CV)

영상을 분석해서 유용한 정보를 추출하는 것(고 수준 영상처리)

**이미지를 입력받아 “정보”나 “판단”을 출력하는 단계**

입력 : 영상 ⇒ 출력 : 서술(특징, 정보, 판단)

## 컴퓨터 그래픽스(CG)

컴퓨터를 이용해 새로운 이미지나 영상을 생성·표현하는 기술 “없는 것을 만들어내는” 분야

입력 : 서술(특징, 정보 등) ⇒ 출력 : 영상

### 연습문제

> 디지털 영상이란?
>

**`아날로그 영상을 이산적인(숫자) 형태로 표현`한 것**

즉, `연속적인 밝기값을 갖는 실제 영상`을 컴퓨터가 처리할 수 있도록

**`좌표(픽셀 단위)** 와 **밝기(그레이 레벨)** 로 변환`한 데이터.

> 저 수준 영상처리와 고 수준 영상처리의 차이
>

| **구분** | **저수준(Low-level) 영상처리** | **고수준(High-level) 영상처리** |
| --- | --- | --- |
| **주요 작업** | 영상의 **개선, 복원, 변환** | **인식, 이해, 해석** |
| **입출력 형태** | 영상 → 영상 | 영상 → 서술(정보, 의미) |
| **예시** | 필터링, 잡음제거, 대비조정, 에지검출 | 객체 인식, 얼굴 인식, 장면 이해, 자율주행 |
| **목적** | 영상 품질 향상 | 영상 의미 분석 |

영상을 통해서 `개선, 복원, 변환 등의 작업을 통해 새 영상을 만들어내는 것`을 **“저수준 영상처리”**라고 하며,

`영상을 통해서 유의미한 분석을하여 정보를 얻는 것`을 **“고수준 영상처리”**라고 한다.

> 카메라의 해상도와 표본화 & 양자화 단계는 어떤 관련이 있는가?
>
- **해상도**: 영상을 구성하는 **픽셀 수(공간적 해상도)** 를 의미.

  카메라 해상도가 높을수록 영상의 **표본화(Sampling)** 단계에서

  더 많은 위치 좌표(픽셀)를 취득.

- **표본화(Sampling)**: 연속적인 영상을 **공간적으로 분할**하는 과정 (x, y 위치를 이산화).

  → 해상도와 직접적인 관련 있음.

- **양자화(Quantization)**: 각 픽셀의 밝기(연속값)를 **유한한 단계(그레이 레벨)** 로 변환하는 과정.

  → 해상도는 그대로 두고 밝기 표현 단계(명암 단계)를 결정.


`영상을 구성하는 총 픽셀 수, 크기`를 해상도라고 하며,

이 `해상도에 맞게 연속적인 흐름의 영상을 공간적(x, y 위치 등)으로 분할` 하는 과정을 표본화라고 한다.

`표본화된 각 픽셀의 밝기를 유한한 단계(값. 255 등)로 변환`하는 과정을 양자화라고 한다.

# 3장

---

## 브로드캐스팅 (BroadCasting)

⇒ 서로 다른 shape을 가진 배열 간의 산술 연산을 가능하게 하는 기법

> 브로드 캐스팅 규칙
>

규칙1. 두 배열의 차원 수가 다르면, 작은 차원을 가진 배열 shape의 앞쪽(왼쪽)을 1로 채움.

```python
# 규칙 1: 두 배열의 차원 수가 다르면, 작은 차원을 가진 배열 shape의 앞쪽(왼쪽)을 1로 채운다.
a = np.arange(24).reshape(2, 3, 4)
print(a)
b = np.arange(12).reshape(3, 4) # b의 차원이 1로 채워짐
print(b)

a + b
```

규칙2. 두 배열의 shape이 다르다면, 차원의 크기가 1인 배열이 다른쪽 배열의 크기(shape)과 일치하도록 수정.

```python
# 규칙 2: 두 배열의 shape이 일치하지 않는다면, 일치하지 않는 차원에서 크기가 1인 배열이 다른 쪽 배열의 크기와 일치하도록 수정한다.
c = np.arange(8).reshape(1, 2, 4) # c의 차원이 2로 바뀜
print(c)
d = np.arange(16).reshape(2, 2, 4)
print(d)

c + d
```

### 연습문제

> 슬라이스 연산자를 사용하는 방법, 10개의 원소 리스트에서 8번째 원소에서 2번째 원소까지 역순으로 출력
>

```python
lst = [1,2,3,4,5,6,7,8,9,10]
print(lst[7:1:-1])  # 8번째~2번째 (간격이 음수면 역순)
```

> 다차원 행렬을 1차원으로 변경하는 방법
>

**np.reshape() 사용법 확인하기**

> 실수형 원소 10개 ndarray 행렬에서 전체 원소의 합과 평균 출력
>

```python
arr = np.array([1.2, 3.4, 5.6, 7.8, 9.0, 2.2, 4.4, 6.6, 8.8, 10.0], np.float32)

print("합:", np.sum(arr))
print("평균:", np.mean(arr))
```

# 4장

---

## ord(’a’)

키 입력 → 아스키 코드 변환

```python
# 키 이벤트 사용

switch_case = {
    # ord() 함수는 문자의 아스키 코드 값을 반환
    ord('a'): "a key is pressed.",
    ord('b'): "b key is pressed.",
    ord('A'): "A key is pressed.",
}

image = np.ones((200, 300), np.float32)
cv2.namedWindow('Keyboard Event')
cv2.imshow('Keyboard Event', image)

# 중요
while True:
	# waitKeyEx()는 해당 시간동안 실제 키 입력을 기다림
    key = cv2.waitKeyEx(100)
    if key == 27: break # esc이면 탈출

    try:
        result = switch_case[key]
        print(result)
    except KeyError:
        result = -1

cv2.destroyAllWindows()
# waitKey() 해당 시간동안 아무 키 입력을 기다림
cv2.waitKey(1)
```

## setMouseCallback(title, onMouse)

```python
# 마우스 이벤트 사용

def onMouse(event, x, y, flags, param):
    if event == cv2.EVENT_LBUTTONDOWN:
        print("왼쪽 버튼 다운:", x, y)
    elif event == cv2.EVENT_LBUTTONUP:
        print("왼쪽 버튼 업:", x, y)
    elif event == cv2.EVENT_RBUTTONDOWN:
        print("오른쪽 버튼 다운:", x, y)
    elif event == cv2.EVENT_RBUTTONUP:
        print("오른쪽 버튼 업:", x, y)

image = np.full((400, 600), 255, np.uint8)
cv2.namedWindow('Mouse Event')

cv2.imshow('Mouse Event', image)
cv2.setMouseCallback('Mouse Event', onMouse)

cv2.waitKey(0)
cv2.destroyAllWindows()
cv2.waitKey(1)
```

## 마우스 드래그로 도형 그리기

```python
# 마우스 드래그로 도형 그리기

def onMouse(event, x, y, flags, param):
    global title, pt                          

    if event == cv2.EVENT_LBUTTONDOWN:
        pt = (x, y)

    elif event == cv2.EVENT_LBUTTONUP:
        cv2.rectangle(image, pt, (x, y), (255,0,0), 2)
        cv2.imshow(title, image)

    elif event == cv2.EVENT_RBUTTONDOWN:
        pt = (x, y)

    elif event == cv2.EVENT_RBUTTONUP:
        dx, dy = (pt[0] - x) / 2, (pt[1] - y)/2
        radius = int(np.sqrt(dx * dx + dy * dy))
        center = int(x+dx), int(y+dy)
        cv2.circle(image, center, radius, (0,0,255), 2)
        cv2.imshow(title, image)                     

image = np.full((300, 500, 3), (255, 255, 255), np.uint8) 

pt = (-1, -1)                                  
title = "Draw Event1"
cv2.imshow(title, image)
cv2.setMouseCallback(title, onMouse) 

cv2.waitKey(0)
cv2.destroyAllWindows()
cv2.waitKey(1)
```

# 5장

---

## flip(img, x)

```python
# 영상 기하학적 변환 (flip)

image = cv2.imread('../images/read_color.jpg', cv2.IMREAD_COLOR)
if image is None: raise Exception("영상파일 읽기 오류")

x_axis = cv2.flip(image, 0)
y_axis = cv2.flip(image, 1)
xy_axis = cv2.flip(image, -1)
rep_image = cv2.repeat(image, 2, 3)
trans_image = cv2.transpose(image)

titles = ['image', 'x_axis', 'y_axis', 'xy_axis', 'rep_image', 'trans_image']
for title in titles:
    cv2.imshow(title, eval(title))

cv2.waitKey(0)
cv2.destroyAllWindows()
cv2.waitKey(1)
```

## 이진화(Binarization)

그레이스케일 영상을 이진 영상으로 만드는 과정

특정한 밝기값(역치값, threshold value)을 기준으로 그보다 어두운 화소는 검은색, 밝은 화소는 흰색으로 변환

1. 특정 임계값을 기준으로 영상 전체를 이진화하는 전역 이진화 방식
2. 영상을 여러 영역으로 나누어서 독립적으로 임계값을 구한 후 해당 영역의 임계값을 기준으로 영역마다 이진화를 하는 방식

### 전역 이진화(+ Otsu 알고리즘)

이진화는 이미지를 두 개의 클래스로 나누는 분류 알고리즘으로 생각할 수 있음

클래스를 나누는데 드는 비용이 적게 드는 상태가 가장 좋은 경우 ⇒ 경계값과 픽셀값의 차가 커야함(나누기 쉬움)

다양한 분포를 가진 영역에서 흩어짐 정도가 가장 작은(분산값)값을 대표값으로 하기 좋다.

> intra-class variance: 최소값 구하는 방식
>

특정 역치값으로 두 클래스로 나누었을 때, 각 클래스가 비율(양)에 비해 흩어짐 정도가 낮을 때의 역치값이 적절함

식 : (클래스0 픽셀 비율)*(클래스0 분산) + (클래스1 픽셀 비율)*(클래스1 분산) ⇒ 중 가장 작을 때

문제점 : 최소값 구하는 방식은 분산을 계산하기 때문에 오래 걸림

> inter-class variance: 최대값 구하는 방식
>

⇒ 두 클래스의 평균 밝기값이 서로 얼마나 멀리 떨어져 있는지를 측정

- 두 클래스가 “얼마나 잘 분리되었나”를 보는 지표
- 내부 분산이 작으면 자연스럽게 클래스 간 분산은 커짐

식 : (클래스0 픽셀 비율)*(클래스1 픽셀 비율)*(클래스0 평균값 - 클래스1 평균값)**2 ⇒ 중 가장 클 때

### 적응형 이진화(adaptive thresholding)

> 영상을 여러 영역으로 나누어서 독립적으로 임계값을 구한 후 해당 영역의 임계값을 기준으로 영역마다 이진화를 하는 방식
>

영상 영역에 따른 이진화 방식으로, 이미지를 여러 영역으로 나누어서 각 영역마다 독립적으로 임계값을 계산 후 thresholding을 진행.

”전체 조명 조건이 일정하지 않은 이미지”에서 **지역별 밝기 차이를 보정하면서 이진화**할 때 사용

`즉, 픽셀마다 “주변 밝기에 맞추어” 이진화 임계값을 달리함.`

block_size x block_size(홀수) 를 기준으로 영역을 구분하고, `평균 방식` 또는 `가우시안 분포 방식`으로 나누어 임계값을 결정한다. 이후 C(가감할 상수)값을 통해 보정한다.

> 평균 방식
>

`주변 블록의 단순 평균 - C`

> 가우시안 분포 방식
>

blockSize 영역의 각 픽셀에 대해 **`중심에 가까울수록 높은 가중치를** 곱해 평균 계산, 중심부의 픽셀 영향이 커짐`

```python
cv2.threshold(img, 127, 255, cv2.THRESH_BINARY)[1]

# cv2로 otsu 알고리즘 사용법
t, t_otsu = cv2.threshold(img, -1, 255, cv2.THRESH_BINARY | cv2.THRESH_OTSU)
print('otsu threshold: ', t)

# inter-class variance
for i in range(255):
    img_s = img_flat[img_flat<=i]
    img_b = img_flat[img_flat>i]
    if len(img_s):
        w1 = len(img_s) / (len(img_s) + len(img_b))
    else:
        continue

    w2 = 1 - w1
    mean_s = np.mean(img_s)
    mean_b = np.mean(img_b)
    vm = w1*w2*(mean_s - mean_b)**2
    
    if vm > var_max:
        var_max, threshold = vm, i
        
        
# 5-8. 적응형 이진화(영상 영역에 따른 이진화)
# cv2.adaptiveThreshold

blk_size = 9
C = 5
img = cv2.imread('../images/sudoku.png', cv2.IMREAD_GRAYSCALE)

ret, th1 = cv2.threshold(img, 0, 255, cv2.THRESH_BINARY | cv2.THRESH_OTSU)

th2 = cv2.adaptiveThreshold(img, 255, cv2.ADAPTIVE_THRESH_MEAN_C, cv2.THRESH_BINARY, blk_size, C)
th3 = cv2.adaptiveThreshold(img, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C, cv2.THRESH_BINARY, blk_size, C)
```

# 6장

---

## 행렬 원소 접근

### LUT

```python
# 영상처리에서 자주 사용하는 연산인 룩업테이블(LUT)로 처리하면 속도가 매우 빨라진다.
def pixel_access3(image):
    lut = [255 - i for i in range(256)]  # 룩업테이블 생성
    lut = np.array(lut, np.uint8)
    # image3 = lut[image]
    image3 = cv2.LUT(image, lut)
    return image3
```

- LUT 언제 쓰면 빨라지는지, 이유는?

⇒  룩업테이블은 픽셀값 변환 결과를 미리 계산하여 테이블 형태로 저장하고, 변환 시 계산 대신 테이블 조회만 수행하므로 **픽셀 수에 비례한 반복 연산을 제거**할 수 있다.

따라서**같은 수식을 모든 픽셀에 적용하는 경우 실행 속도가 매우 빨라진다.**

### 화소 밝기 변환 (cv방식과 np방식의 차이)

```python
# 차이점 : OpenCV 함수와 numpy array 연산
# OpenCV 함수는 saturation 방식으로 연산을 수행하고, numpy array 연산은 modulo 방식으로 연산을 수행한다.(주의)

# OpenCV 함수 이용
dst1 = cv2.add(image, 200)                  # 영상 밝게 saturation 방식 
dst2 = cv2.subtract(image, 200)             # 영상 어둡게

# numpy array 이용
dst3 = image + 100                          # 영상 밝게 modulo 방식 pixel(240) + 200 -> 184 ..why?( 440 % 256 = 184)
dst4 = image - 100                          # 영상 어둡게           pixel(240) - 250 -> 246 ..why? (-10 % 256 = 246) 따라서 밝게하려고 + 하너가 어둡게 하려고 - 할 때 
                                            #                     반대되는 현상 또는 예기치 않은 현상이 발생해 오류처럼 보일 수 있음
```

1. OpenCV 방식은 add로 더하거나 subtract로 빼도 최소0, 최대 255 값을 유지한다.(saturation 방식)
2. Numpy 방식은 modulo 방식을 사용하는데 원리는 아래와 같음

> pixel(240) + 200 -> 184 ..why?( 440 % 256 = 184)
pixel(240) - 250 -> 246 ..why? ( -10 % 256 = 246)
>

**따라서 밝게하려고 + 하거나, 어둡게 하려고 - 할 때 반대되는 현상 또는 예기치 않은 현상이 발생해 오류처럼 보일 수  있음**

### 영상 합성 cv2.addWeighted()

```python
# 합성 비율 지정
alpha, beta = 0.4, 0.6

add_img1 = cv2.add(image1, image2)  # OpenCV 함수 이용
print('add_image1:', add_img1.dtype) # 하얗게 나오는 이유 확인하기

add_img2 = cv2.add(image1 * alpha, image2 * beta)  # OpenCV 함수 이용
print('add_image2:', add_img2.dtype) # float이며, 값의 범위가 넘어갈 것
add_img2 = np.clip(add_img2, 0, 255).astype(np.uint8)  # numpy array 연산 후 clip

add_img3 = cv2.addWeighted(image1, alpha, image2, beta, -50)  # OpenCV 함수 이용
print('add_image3:', add_img3.dtype)
```

### 명암 대비

명암 대비란 상이한 두 밝기의 경계에서 서로 영향을 미쳐 그 차이가 강조되는 현상.

명암 대비가 크다는 것은 어두운 부분은 어둡게, 밝은 부분은 밝게 된다는 것.

명암 대비를 늘리려면 1.0 이상의 값을 곱해주고(밝고 어두운 영역 차가 커짐), 줄이려면 1.0 이하의 값을 곱하면 됨

## 히스토그램

### 흑백

```python
def draw_histo(hist, shape=(200, 256)):
    hist_img = np.full(shape, 255, np.uint8)
		
		# 히스토그램 값들을 hist_img 크기에 맞게 정규화(축소 또는 확대)하는 과정
    cv2.normalize(hist, hist, 0, shape[0], cv2.NORM_MINMAX) # 히스토그램 정규화
    gap = hist_img.shape[1]/hist.shape[0]             # 한 계급 너비

    for i, h in enumerate(hist.flat):
        x = int(round(i * gap))                         # 막대 사각형 시작 x 좌표
        w = int(round(gap))
        cv2.rectangle(hist_img, (x, 0, w, int(h)), 0, cv2.FILLED)
    return   cv2.flip(hist_img, 0)                        # 영상 상하 뒤집기 후 반환

image = cv2.imread("../images/draw_hist.jpg", cv2.IMREAD_GRAYSCALE)  # 영상 읽기
if image is None: raise Exception("영상 파일 읽기 오류")

# 영상의 0번 채널(흑백) 대상으로 256개의 구간을 0~255범위까지 보겠다는 얘기    
hist = cv2.calcHist([image], [0], None, [256], [0, 256])
hist_img = draw_histo(hist)

titles = ["image", "hist_img"]
for t in titles:
    cv2.imshow(t, eval(t))

cv2.waitKey(0)
cv2.destroyAllWindows()
cv2.waitKey(1)
```

### 컬러

HSV 색공간은 색상(H), 채도(S), 명도(V)를 분리해 표현하기 때문에

H 각도값을 통해서 색상만을 독립적으로 조절하거나 특정 색을 추출하는 것이 쉬움.

반면 BGR은 세 채널에 `색상·밝기` 정보가 `섞여 있어 색 분석에는 부적합.`

openCV에서는 360도를 180도로 압축해서 사용

# Color 영상

---

## 색

- 색은 물체에서 반사되는 빛의 성질에 의해 결정됨
- 빛은 전자기파의 일종, 매질이 필요없는 파동으로 다양한 파장임
- 파장에 따라 빛은 가시광선, 적외선, 자외선, 감마선
- 가시광선은 약 380nm ~ 780nm
- 주파수 = 1/주기
- 광속 = 주파수 * 파장

## 눈의 구조

빛은 망막을 통해 흡수되며 망막에는 `원추세포`와 `간상세포`가 존재

- **원추 세포는 `색상을 구분`**
- **간상 세포는 `명암을 구분`**

원추 세포는 다시 빨강, 초록, 파랑에 반응하는 세가지 종류 세포로 구분됨(명확히 그 색만 구분하는 것은 아니고, 상대적 민감도가 셋 다 다른 것.)

**사람은 `간상세포가 많기 때문에 명암을 색상보다 더 잘 구분`**

홍채를 통해 빛이 들어오면 수정체에서 빛이 굴절되고, 망막에 상이 맺히게 된다.

망막 뒷 부분으로 시신경이 이어져있어 사람이 물체를 인식할 수 있다.(시신경이 없는 부분에 상이 맺히면 보이지 않는다)

- 빛의 양 조절: 홍채
- 초점 조절: 수정체

## 삼색 정합

세 가지 빛을 어느 정도의 양으로 혼합해야 하는지 알 수 있으면 모든 색의 표현이 가능

`빨, 초, 파 세 가지 파장을 사용하여 표현 가능한 색을 조합하는 것을 삼색 정합`이라고 함

## 색의 표현 → 컬러 모델

### RGB 모델

R, G, B, 원점은 검은색(0, 0, 0) 최대값은 흰색(255, 255, 255)

`빛의 성질을 이용한 가산 모델`

> 컬러 깊이: 컬러값을 위해 사용된 비트수
>

1비트 컬러 : Black & White

8비트 컬러(1) GrayScale: 회색조 컬러

24비트 컬러 : 눈이 구별할 수 있는 컬러보다 많음

PNG 형식 → 48비트 컬러 사용

8비트 컬러(인덱스 컬러)

직접 컬러 → 256가지 색상 (하지만 너무 제한적)

256개 특정 컬러 팔레트는 `실제 24비트 컬러 팔레트에서 필요한 값만 참고`하는 방식

- 시스템 팔레트 : 일반적으로 자주 쓰는 색상들을 모아둔 팔레트

⇒ 일부 컬러는 팔레트에서 누락되고, 가장 가까운 컬러값으로 대체할 수 있다(하지만 포스토 효과가 발생할 수 있음)

⇒ ⇒ **`디더링`**: 하나의 컬러 영역을 여러 컬러 패턴으로 대체해서 해결할 수 있음

> BMP 파일 이해하기
>

BMP 파일 구조

비트맵 파일 헤더, 비트맵 정보 헤더, 색상 테이블/팔레트, +픽셀 데이터

헤더의 정보를 통해서 픽셀 데이터를 나타낸다.

### CMYK 모델

Cyan(청록), Magenta(심홍), Yellow(노랑)

물감, 잉크의 성질을 이용한 감산 모델(감법 원색) / ex) 흰색 - 빨간색 = 시안색

CMY는 RGB의 보색 / ex) 시안색은 빨간색의 보색

> 문제점
>
1. 정확한 보색의 빛만 흡수하는 잉크 제작 불가
2. 검은색을 보기 위해서는 세 가지 잉크를 칠해야함(비효율적)

⇒ `검은색(Kappa)을 실제로는 추가`

- 색역이 RGB보다 작음

### HSV 모델

Hue(색상), Saturation(채도), Value(명도, 밝기값)

인간의 시각 모델과 유사한 컬러 모델

일반적으로 색상을 표현할 때 (11, 13, 156) 이렇게 표현하지 않고,

“밝지만 탁한 푸른색”과 같이 표현하는데, 이를 표현한 모델이다.

`Hue(색상)`은 색의 주 파장을 구분하는 특징

`Saturation(채도)`는 색의 순수성(연한, 진한 등)을 구분하는 특징(중앙 기준 퍼짐 정도)

`Value(명도, 밝기값)`은 밝고 어두운 정도를 구분하는 값(위, 아래)

⇒ 실린더 좌표라는 원뿔 형태를 사용하여 표현함


Hue(각도)에 따라 색상을 나눌 수 있고, 중앙을 기준으로 퍼질 수록 Saturation(채도) 증가, 아래로 내려갈 수록 Value(명도, 밝기값)이 낮아짐

### YUV 모델

컬러에서 밝기 정보는 분리할 수 있다.

밝기는 RGB 각 요소의 빛이 얼마나 많이 방출되는가에 따라 결정됨

Y(밝기값) = (R+G+B) / 3

Y값을 가지고 R값과 B값을 안다면 G값을 구할 수 있다.

따라서 흑백, 컬러 모두 표현하고 싶을 때 Y값 정보를 일단 보내고, 컬러가 필요할 때는 R, B 값을 보내어 표현한다.

아날로그 TV에서는 Y와 가중치를 갖는 색차 U(파란 성분의 정도), V(빨간 성분의 정도)가 사용된다.

디지털 TV에서는 Y와 Blue(Color), Red(Color)가 사용된다.