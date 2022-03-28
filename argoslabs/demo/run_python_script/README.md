# Run Python Script (v3.507.1938) 사용법

# Run Python Script 소개

`Run Python Script` 는 파이썬 스크립트 파일을 실행할 수 있는 플러그인 입니다. 다른 플러그인과 마찬가지로 시나리오에 포함되어 변수 및 각종 옵션을 사용하여 실행하며, 결과 값 역시 사용자가 작성한 스크립트에서 출력한 데이터를 이용할 수 있습니다. `Run Python Script` 플러그인으로 `official`이나 `private` 저장소에 등록하지 않아도 사용할 수 있는 커스텀 플러그인을 작성하는 것과 비슷합니다.  

 

아래와 같은 경우 사용을 권장합니다.

- STU 오퍼레이션으로 작성하기에는 너무 복잡해지는 구간
- 아직 플러그인으로 만들어지지 않은 기능이 필요할때
- 파이썬 고유의 기능을 이용할 필요가 있을 때

아래와 같은 기능을 제공합니다.

- 파이썬 3.7 스크립트 실행
- 파이썬 모듈 설치
- ARGOS LABS RPA `Official` Plugins 설치 및 사용

추가 고려중

- 파이썬 스크립트 파일 외, STU 프로퍼티 에디터에서 코드입력기 제공하여 시나리오에 직접 코드 저장

# Run Python Script 설치

`Run Python Script` 를 이용하여 스크립트를 작성하고 테스트하기 위해 매번 STU와 PAM을 사용할 필요는 없습니다.  `파이썬 가상환경` 을 만들고 `Run Python Script` 와 필요한 파이썬 패키지 및 `ARGOS LABS Plugin`을 설치하여 작성한 파이썬 스크립트를 테스트 해볼 수 있습니다.

## Python 3.7 위치

- 모든 커맨드 입력은 `cmd` 를 기준으로 합니다.

`PAM` 을 설치하면 Python3.7이 포함되어 있으며, 환경변수 `PATH`에 경로를 추가 시키면 편하게 사용할 수 있습니다. 

```bash
%HOMEPATH%\.argos-rpa.venv\Python37-32\python.exe --version
Python 3.7.3
```

- 아래 명령어 부터는 환경변수 `PATH` 에 Python3.7의 경로가 설정되어 있다고 가정합니다.

## 가상환경 생성 및 적용

```bash
# 현재 위치한 디렉토리에 test_venv 라는 가상환경을 생성 
python.exe -m venv test_venv
test_venv\Scripts\activate.bat
(test_venv) PATH>
```

## Run Python Script 설치

`STU`에서 `Run Python Script` 오퍼레이션을 추가하고 실행했을 경우 설치되는 플러그인과 같은 것이 설치된다.

```bash
pip install argoslabs.demo.run_python_script -i https://pypi-test.argos-labs.com/simple --trusted-host pypi-test.argos-labs.com -U --no-cache-dir
```

# Run Python Script 실행

```bash
positional arguments:
  python_script         python script

optional arguments:
  -i INSTALL, --install INSTALL
  -a ARGUMENTS, --arguments ARGUMENTS
  -k KEY_ARGUMENTS, --key-arguments KEY_ARGUMENTS
  -c, --code
```

## Positional Arguments

첫 번째 인자 값으로 사용자가 작성한 `Python Script File` 을 반드시 써야 합니다.

## Optional Arguments

### Installing Python Package

- 파이썬 스크립트에서 사용될 외부 패키지를 설치
- 설치될 패키지 위치는 `alabs.ppm` 이 플러그인을 위해 생성한 가상환경
- 설치 실패했을 경우에만 `stderr` 로 에러 메세지를 출력
- 한 번 설치에 성공하면 다른 가상환경으로 실행되지 않는 한 재설치하지 않음
- 한 개의 패키지당 한 개의 옵션 사용
- 설치할 특정 버전 명시 가능

`-i`, `--install` 

### Arguments (Sequential Arguments)

- 순서가 정해져 있는 인자 값
- 한 개의 인자 값 마다 한 개의 옵션 사용

`-a`, `--arguments`

### Keyword Arguments

- 순서가 정해져 있지 않음
- Key=Value 형식으로 사용
- 한 개의 인자 값 마다 한 개의 옵션 사용

`-k`, `--key-arguments`

### Printing Code

- 파이썬 스크립트를 작성하는 단계에서 쉽게 테스트, 디버깅하기 위해 사용
- 단독 실행할 수 있는 코드를 출력
- 이 옵션 사용시 파이썬 스크립트를 실행하지 않음

`-c`, `--code`

### 예제

```bash
# -a 입력된 순서대로 인자 값을 사용
# -k 키워드 인자 값 사용
# -i 옵션을 중복적으로 입력하여 여러 패키지 설치 가능
python -m argoslabs.demo.run_python_script sample_script_2.py -a1 -a2 -a3 -k abc=2 -i requests -i modbusclc==0.3.0
```

## 실행하기

### 코드 생성

```bash
python -m argoslabs.demo.run_python_script sample_script_2.py -a1 -a2 -a3 -k abc=2 --code

# ===== 사용자가 작성한 코드가 추가 됨
def main(a, b, c, abc):
    return a, b, c, abc

# ===== 단독 실행을 위해 필요한 코드 생성

import sys
import traceback
import shlex
import subprocess    

def _run_plugin(cmd):
    with subprocess.Popen(shlex.split(cmd), shell=False, stdout=subprocess.PIPE, stderr=subprocess.PIPE) as proc:
        out = proc.stdout.read().decode('utf-8')
        err = proc.stderr.read().decode('utf-8')
    return out, err

def run_plugin(cmd):
    out, err = _run_plugin(cmd)
    if err:
        raise Exception(f'Plugin {err} >> Command: {cmd}')
    return out
    
if __name__ == '__main__':
		# 사용자가 입력한 변수를 아래 main 함수 호출시 사용되도록 자동 생성
    print(main("1", "2", "3", abc="2"))
```

### `Run Python Script` 플러그인을 이용하여 실행

```bash
# 생성된 코드를 바로 실행
python -m argoslabs.demo.run_python_script sample_script_2.py -a1 -a2 -a3 -k abc=2 --code | python
```

### 파일로 저장

```bash
# test.py 라는 이름으로 저장
python -m argoslabs.demo.run_python_script sample_script_2.py -a1 -a2 -a3 -k abc=2 --code > test.py
```

# main() - 메인함수 파라메터

## 정적인 파라메터 입력 방법

### 정해진 개수 또는 키워드 입력 값을 요구할 때

```python
# -a 1 -a 2 -a 3 -a 4 -k ab=one -k cd=two
def main(a, b, c, d, ab, de):
    print(args) # (1, 2, 3, 4)
		print(kwargs) # {'ab': 'one', 'de': 'two'}
    return None

```

- 키워드 파라메터는 반드시 `k=v` 형태를 가져야 한다.  `v` 는 없을 경우 빈 스트링 처리
    - abc=123  == `{'abc': '123'}`
    - abc= == `{'abc':''}`

```python
# -a 1 -a 2 -a 3 -k c=4 -k ab=one -k cd=two
def main(a, b, c, d, ab, ce):
		#
    print(args) # (1, 2, 3)
		print(kwargs) # {'c': '4', 'ab': 'one', 'de': 'two'}
    return None
```

## 유연한 파라메터 입력 방법

입력 받는 파라메터가 가변적일 경우 사용할 수 있다.

유연한 파라메터 입력 방식을 사용할 경우 주의사항

- args 의 개수를 넘어서는 인덱스를 사용해서는 안된다.
- kwargs 에 존재하지 않는 키를 불러서는 안된다.

```python
# -a 1 -a 2 -a 3 -a 4 -k ab=one -k de=two
def main(*args, **kwargs):
		a, b, c, d = args  
		# a=1, b=2, c=3, d=4
		# 총 네 개의 변수에 차례대로 대입. args에 네 개 이하의 값이 있다면 에러
		
		e = kwargs['ab']  # e='one'
		# f = kwargs['ff'] <- ff는 존재하지 않는 키워드 값이므로 에러
		f = kwargs.setdefault('ff', 'two')
		# kwargs는 dict형이며, dict의 setdefault를 이용하여 키워드가 없을 경우 기본 값 반환
    return None
```

# 스크립트 작성시 도움될 모듈

## 기본 모듈

설치가 필요없는 모듈 모음

### datetime

날짜와 시간의 다양한 형태 데이터를 파싱하고 계산할 수 있습니다.

```python
import datetime

def main(dt: str):
    """
    Converting string type date to unix time-stamp
    dt: 2021-03-26 11:30:20
    return 1616725820.0
    """
    d = datetime.datetime.strptime(dt, "%Y-%m-%d %H:%M:%S")
    return d.timestamp()
```

### math

복잡한 수학 계산이 필요할때 사용할 수 있습니다.

[https://docs.python.org/ko/3/library/math.html](https://docs.python.org/ko/3/library/math.html)

### random

다양한 방법으로 임의의 수, 문자를 선택 할 수 있습니다.

[https://docs.python.org/ko/3/library/random.html](https://docs.python.org/ko/3/library/random.html)

### csv

csv 포맷의 데이터를 읽고 쓰기를 할 수 있는 쉬운 사용법의 함수를 지원합니다.

[https://docs.python.org/ko/3/library/csv.html?highlight=csv](https://docs.python.org/ko/3/library/csv.html?highlight=csv)

### SQLite3

초경량 DB로 많이 사용되며 입력받은 데이터를 저장하며 쿼리하여 쉽게 추출할 수 있습니다. CSV 파일로 내보내기가 가능합니다. 

[https://docs.python.org/ko/3/library/sqlite3.html?highlight=sqlite3#module-sqlite3](https://docs.python.org/ko/3/library/sqlite3.html?highlight=sqlite3#module-sqlite3)

### re

정규식 일치 연산을 사용할 수 있습니다. 

[https://docs.python.org/ko/3/library/re.html?highlight=re#module-re](https://docs.python.org/ko/3/library/re.html?highlight=re#module-re)

### JSON

JSON 형태의 데이터 포맷을 다룹니다.

[https://docs.python.org/ko/3/library/json.html](https://docs.python.org/ko/3/library/json.html)

## 외부 모듈

### requests

HTTP 프로토콜 핸들을 제공

[https://requests.readthedocs.io/en/master/](https://requests.readthedocs.io/en/master/)

### BeautifulSoup

HTML 파싱 

### PyYaml

Yaml 포맷은 JSON과 1:1 매칭되는 포맷입니다.

[https://pypi.org/project/PyYAML/](https://pypi.org/project/PyYAML/)

### redis

redis의 클라이언트 모듈입니다. 

### Pillow

이미지 크기 조절, 합성등 이미지 처리를 할 수 있는 모듈입니다.

### Torpy

토르 브라우져 파이썬 API. 몇 개국을 우회하여 자신의 정보를 감춘다.