# AutoScraper로 Webスクレイピング하기

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/)

이 가이드는 Python에서 AutoScraper를 사용하여 Webスクレイピング하는 방법을 설명합니다:

- [AutoScraper란 무엇입니까?](#what-is-autoscraper)
- [프로젝트 설정](#setting-up-a-project)
- [대상 웹사이트 선택](#selecting-a-target-website)
- [AutoScraper로 간단한 데이터 스크레이핑](#scraping-simple-data-with-autoscraper)
- [복잡한 설계의 웹사이트에서 데이터 추출](#extracting-data-from-websites-with-a-complex-design)
- [AutoScraper 사용 시 일반적인 과제](#common-challenges-with-autoscraper)

## What Is AutoScraper?

[AutoScraper](https://github.com/alirezamika/autoscraper)는 예시 쿼리를 기반으로 웹사이트에서 데이터를 자동으로 식별하고 추출하여, 수동 HTML 검사 필요성을 제거함으로써 Webスクレイピング을 단순화하는 Python 라이브러리입니다. 최소한의 설정으로 동적 웹사이트도 효율적으로 처리합니다. 동적 웹사이트 스クレイピング에 대해 더 알아보려면 [여기](https://brightdata.co.kr/blog/how-tos/scrape-dynamic-websites-python)를 확인하십시오.

## Setting up a Project

Python 3 또는 그 이후 버전을 설치한 다음, 프로젝트 디렉터리를 생성하고 이동합니다:

```bash
mkdir auto-scrape && cd auto-scrape
```

가상 환경을 생성합니다:

```bash
python -m venv env
```

그다음 활성화합니다. Linux 및 macOS에서는 다음을 실행합니다:

```bash
source env/bin/activate
```

Windows에서는 다음을 실행합니다:

```powershell
venv\Scripts\activate
```

`autoscraper` 및 `pandas` 라이브러리를 설치합니다:

```bash
pip install autoscraper pandas
```

## Selecting a Target Website

공개 웹사이트를 스크레이핑할 때는 사이트의 Terms of Service(ToS) 또는 `robots.txt` 파일을 확인하여 스크레이ピング이 허용되는지 확인해야 합니다.

이 가이드는 스크레이핑 도구 테스트를 위해 설계된 초보자 친화적 샌드박스인 [Scrape This Site’s Countries of the World: A Simple Example page](https://www.scrapethissite.com/pages/simple/)에서 데이터를 스크레이핑하는 것을 가정합니다.

이 페이지는 구조가 단순하여 데이터 추출의 기본을 학습하기에 훌륭한 출발점입니다. 기본 개념에 익숙해지면 더 복잡한 예시인 [Hockey Teams: Forms, Searching, and Pagination page](https://www.scrapethissite.com/pages/forms/)로 넘어가겠습니다.

## Scraping Simple Data with AutoScraper

다음 스크립트를 사용하여 국가 목록과 함께 수도, 인구, 면적을 스크레이핑합니다:

```python
# 1. Import dependencies
from autoscraper import AutoScraper
import pandas as pd

# 2. Define the URL of the site to be scraped
url = "https://www.scrapethissite.com/pages/simple/"

# 3. Instantiate the AutoScraper
scraper = AutoScraper()

# 4. Define the wanted list by using an example from the web page
# This list should contain some text or values that you want to scrape
wanted_list = ["Andorra", "Andorra la Vella", "84000", "468.0"]

# 5. Build the scraper based on the wanted list and URL
scraper.build(url, wanted_list)

# 6. Get the results for all the elements matched
results = scraper.get_result_similar(url, grouped=True)

# 7. Display the keys and sample data to understand the structure
print("Keys found by the scraper:", results.keys())

# 8. Assign columns based on scraper keys and expected order of data
columns = ["Country Name", "Capital", "Area (sq km)", "Population"]

# 9. Create a DataFrame with the extracted data
data = {columns[i]: results[list(results.keys())[i]] for i in range(len(columns))}
df = pd.DataFrame(data)

# 10. Save the DataFrame to a CSV file
csv_filename = 'countries_data.csv'
df.to_csv(csv_filename, index=False)

print(f"Data has been successfully saved to {csv_filename}")
```

위 스크립트는 필요한 라이브러리인 AutoScraper와 pandas를 임포트하는 것으로 시작합니다. 다음으로 대상 웹사이트의 URL을 정의한 뒤 AutoScraper 인스턴스를 생성합니다.

여기서 AutoScraper의 강점이 드러납니다. XPath나 CSS selector와 같은 명시적 지시가 필요한 전통적인 스크레이퍼와 달리, AutoScraper는 예시 데이터를 제공할 수 있습니다. 페이지에서 요소가 어디에 있는지 지정하는 대신, 추출하려는 샘플 값을 몇 개 나열하기만 하면 됩니다. `wanted_list`라고 불리는 이 리스트에는 국가명, 수도, 인구, 면적과 같은 대표 데이터 포인트가 포함됩니다.

`wanted_list`를 설정하면 제공된 URL과 샘플 데이터를 사용하여 스크레이퍼를 빌드합니다. 그러면 AutoScraper가 웹페이지를 분석하고 패턴을 식별한 후 스크레이핑 규칙을 생성합니다. 이 규칙을 통해 스크레이퍼는 동일한 또는 다른 웹페이지에서 유사한 데이터를 인식하고 추출할 수 있습니다.

스크립트의 아래쪽에서는 `get_result_similar` 메서드를 호출하여 AutoScraper가 학습한 패턴과 일치하는 모든 데이터를 가져옵니다. 스크레이퍼가 데이터 구조를 어떻게 해석하는지 더 잘 이해하기 위해, 스크립트는 추출된 데이터와 연결된 rule ID를 출력합니다. 출력은 다음과 유사해야 합니다:

```
Keys found by the scraper: dict_keys(['rule_4y6n', 'rule_gghn', 'rule_a6r9', 'rule_os29'])
```

주석 8과 9는 컬럼 이름을 정의하고 추출된 데이터를 pandas DataFrame으로 구조화합니다. 주석 10은 이 데이터를 CSV 파일로 저장합니다.

스크립트(`python script.py`)를 실행하면, 프로젝트 디렉터리에 `countries_data.csv` 파일이 생성되며 다음과 같은 내용이 포함됩니다:

```csv
Country Name,Capital,Area (sq km),Population
Andorra,Andorra la Vella,84000,468.0
United Arab Emirates,Abu Dhabi,4975593,82880.0
...246 collapsed rows
Zambia,Lusaka,13460305,752614.0
Zimbabwe,Harare,11651858,390580.0
```

## Extracting Data from Websites with a Complex Design

이전 기법은 많은 유사 값이 있는 테이블이 포함된 [Hockey Teams page](https://www.scrapethissite.com/pages/forms/)와 같은 복잡한 웹사이트에서는 어려움을 겪을 수 있습니다. 동일한 방법으로 팀 이름, 연도, 승수 및 기타 필드를 추출해 보면서 어떤 문제가 발생하는지 확인해 보십시오.

다행히 AutoScraper는 빌드 단계에서 불필요한 rule을 가지치기(pruning)하여 모델을 정교화하는 것을 지원합니다. 방법은 다음과 같습니다:

```python
from autoscraper import AutoScraper
import pandas as pd

# Define the URL of the site to be scraped
url = "https://www.scrapethissite.com/pages/forms/"

def setup_model():

    # Instantiate the AutoScraper
    scraper = AutoScraper()

    # Define the wanted list by using an example from the web page
    # This list should contain some text or values that you want to scrape
    wanted_list = ["Boston Bruins", "1990", "44", "24", "0.55", "299", "264", "35"]

    # Build the scraper based on the wanted list and URL
    scraper.build(url, wanted_list)

    # Get the results for all the elements matched
    results = scraper.get_result_similar(url, grouped=True)

    # Display the data to understand the structure
    print(results)

    # Save the model
    scraper.save("teams_model.json")

def prune_rules():
    # Create an instance of Autoscraper
    scraper = AutoScraper()
    
    # Load the model saved earlier
    scraper.load("teams_model.json")

    # Update the model to only keep necessary rules
    scraper.keep_rules(['rule_hjk5', 'rule_9sty', 'rule_2hml', 'rule_3qvv', 'rule_e8x1', 'rule_mhl4', 'rule_h090', 'rule_xg34'])

    # Save the updated model again
    scraper.save("teams_model.json")
    
def load_and_run_model():
    # Create an instance of Autoscraper
    scraper = AutoScraper()
    
    # Load the model saved earlier
    scraper.load("teams_model.json")

    # Get the results for all the elements matched
    results = scraper.get_result_similar(url, grouped=True)

    # Assign columns based on scraper keys and expected order of data
    columns = ["Team Name", "Year", "Wins", "Losses", "Win %", "Goals For (GF)", "Goals Against (GA)", "+/-"]

    # Create a DataFrame with the extracted data
    data = {columns[i]: results[list(results.keys())[i]] for i in range(len(columns))}
    df = pd.DataFrame(data)

    # Save the DataFrame to a CSV file
    csv_filename = 'teams_data.csv'
    df.to_csv(csv_filename, index=False)

    print(f"Data has been successfully saved to {csv_filename}")

# setup_model()
# prune_rules()
# load_and_run_model()
```

이 스크립트는 `setup_model`, `prune_rules`, `load_and_run_model`의 세 가지 메서드를 정의합니다.

`setup_model` 메서드는 스크레이퍼 인스턴스를 생성하고 `wanted_list`를 정의한 다음 스크레이퍼를 빌드합니다. 이후 대상 URL에서 데이터를 스크레이핑하고, 수집된 rule ID를 출력하며, 모델을 `teams_model.json`으로 프로젝트 디렉터리에 저장합니다.

스크립트를 실행하려면 스크립트에서 `# setup_model()` 줄의 주석을 해제하고 파일(예: `script.py`)을 저장한 뒤 `python script.py`로 실행하십시오.

출력은 다음과 같이 표시됩니다:

```
{'rule_hjk5': ['Boston Bruins', 'Buffalo Sabres', 'Calgary Flames', 'Chicago Blackhawks', 'Detroit Red Wings', 'Edmonton Oilers', 'Hartford Whalers', 'Los Angeles Kings', 'Minnesota North Stars', 'Montreal Canadiens', 'New Jersey Devils', 'New York Islanders', 'New York Rangers', 'Philadelphia Flyers', 'Pittsburgh Penguins', 'Quebec Nordiques', 'St. Louis Blues', 'Toronto Maple Leafs', 'Vancouver Canucks', 'Washington Capitals', 'Winnipeg Jets', 'Boston Bruins', 'Buffalo Sabres', 'Calgary Flames', 'Chicago Blackhawks'], 'rule_uuj6': ['Boston Bruins', 'Buffalo Sabres', 'Calgary Flames', 'Chicago Blackhawks', 'Detroit Red Wings', 'Edmonton Oilers', 'Hartford Whalers', 'Los Angeles Kings', 'Minnesota North Stars', 'Montreal Canadiens', 'New Jersey Devils', 'New York Islanders', 'New York Rangers', 'Philadelphia Flyers', 'Pittsburgh Penguins', 'Quebec Nordiques', 'St. Louis Blues', 'Toronto Maple Leafs', 'Vancouver Canucks', 'Washington Capitals', 'Winnipeg Jets', 'Boston Bruins', 'Buffalo Sabres', 'Calgary Flames', 'Chicago Blackhawks'], 'rule_9sty': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_9nie': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_41rr': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_ufil': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_ere2': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_w0vo': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_rba5': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_rmae': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_ccvi': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_3c34': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_4j80': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_oc36': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_93k1': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_d31n': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_ghh5': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_5rne': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_4p78': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_qr7s': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_60nk': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_wcj7': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_0x7y': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_2hml': ['44', '31', '46', '49', '34', '37', '31', '46', '27', '39', '32', '25', '36', '33', '41', '16', '47', '23', '28', '37', '26', '36', '31', '31', '36'], 'rule_swtb': ['24'], 'rule_e8x1': ['0.55', '14', '0.575', '0.613', '-25', '0', '-38', '0.575', '-10', '24', '8', '-67', '32', '-15', '0.512', '-118', '0.588', '-77', '-72', '0', '-28', '-5', '-10', '-9', '21'], 'rule_3qvv': ['24', '30', '26', '23', '38', '37', '38', '24', '39', '30', '33', '45', '31', '37', '33', '50', '22', '46', '43', '36', '43', '32', '37', '37', '29'], 'rule_n07w': ['24', '30', '26', '23', '38', '37', '38', '24', '39', '30', '33', '45', '31', '37', '33', '50', '22', '46', '43', '36', '43', '32', '37', '37', '29'], 'rule_qmem': ['0.55', '0.388', '0.575', '0.613', '0.425', '0.463', '0.388', '0.575', '0.338', '0.487', '0.4', '0.312', '0.45', '0.412', '0.512', '0.2', '0.588', '0.287', '0.35', '0.463', '0.325', '0.45', '0.388', '0.388', '0.45'], 'rule_b9gx': ['264', '278', '263', '211', '298', '272', '276', '254', '266', '249', '264', '290', '265', '267', '305', '354', '250', '318', '315', '258', '288', '275', '299', '305', '236'], 'rule_mhl4': ['299', '292', '344', '284', '273', '272', '238', '340', '256', '273', '272', '223', '297', '252', '342', '236', '310', '241', '243', '258', '260', '270', '289', '296', '257'], 'rule_24nt': ['264', '278', '263', '211', '298', '272', '276', '254', '266', '249', '264', '290', '265', '267', '305', '354', '250', '318', '315', '258', '288', '275', '299', '305', '236'], 'rule_h090': ['264', '278', '263', '211', '298', '272', '276', '254', '266', '249', '264', '290', '265', '267', '305', '354', '250', '318', '315', '258', '288', '275', '299', '305', '236'], 'rule_xg34': ['35', '14', '81', '73', '-25', '0', '-38', '86', '-10', '24', '8', '-67', '32', '-15', '37', '-118', '60', '-77', '-72', '0', '-28', '-5', '-10', '-9', '21']}
```

이 출력은 대상 웹사이트에서 `get_result_similar` 호출을 통해 AutoScraper가 수집한 데이터를 보여줍니다. AutoScraper는 관계를 추론하고 데이터를 rule로 그룹화하려고 시도하기 때문에 중복이 포함됩니다. 올바르게 그룹화되면, 유사한 사이트에서도 데이터를 쉽게 추출할 수 있습니다.

그러나 이 사이트는 숫자가 많아 AutoScraper가 어려움을 겪으며, 그 결과 잘못된 상관관계가 다수 발생하고 중복이 포함된 큰 데이터셋이 생성됩니다.

계속 진행하려면 데이터를 분석하고 올바른 데이터(즉, 올바른 순서로 하나의 컬럼 데이터와 일치하는 데이터)를 가진 rule을 선택해야 합니다. 이 경우, 다음 rule들이 올바른 데이터를 포함하고 있었습니다(각 rule에 대상 페이지의 행 수에 해당하는 25개 데이터 포인트가 들어 있는지 검증하여 판단했습니다):

```
['rule_hjk5', 'rule_9sty', 'rule_2hml', 'rule_3qvv', 'rule_e8x1', 'rule_mhl4', 'rule_h090', 'rule_xg34']
```

`prune_rules` 메서드를 업데이트하십시오. 그런 다음 스크립트에서 `setup_model()` 줄을 주석 처리하고 `prune_rules()` 줄의 주석을 해제하십시오. 실행하면 `teams_model.json`에서 모델을 로드하고 불필요한 rule을 제거한 뒤 업데이트된 모델을 저장합니다. `teams_model.json`의 내용을 확인하여 저장된 rule을 검증할 수 있습니다.

다음으로 `load_and_run_model` 메서드를 실행하려면, `prune_rules` 줄을 주석 처리하고 `load_and_run_model` 줄의 주석을 해제한 뒤 스크립트를 다시 실행하십시오. 그러면 프로젝트 디렉터리의 `teams_data.csv`에 올바른 데이터가 추출되어 저장되며, 다음 출력도 표시됩니다:

```
Data has been successfully saved to teams_data.csv
```

성공적으로 실행한 뒤 `teams_data.csv` 파일은 다음과 같습니다:

```csv
Team Name,Year,Wins,Losses,Win %,Goals For (GF),Goals Against (GA),+/-
Boston Bruins,1990,44,0.55,24,299,264,35
Buffalo Sabres,1990,31,14,30,292,278,14
...21 more rows
Calgary Flames,1991,31,-9,37,296,305,-9
Chicago Blackhawks,1991,36,21,29,257,236,21
```

이 글에서 개발된 모든 코드는 [this GitHub repo](https://github.com/krharsh17/auto-scrape)에서 확인할 수 있습니다.

## Common Challenges with AutoScraper

AutoScraper는 작은 データセット과 뚜렷한 데이터 포인트가 있는 단순한 사용 사례에 매우 적합합니다. 하지만 테이블을 스크레이핑하는 것과 같은 더 복잡한 시나리오에서는 번거로울 수 있습니다. 또한 JavaScript 렌더링을 지원하지 않으므로, [Splash](https://pypi.org/project/splash/), Selenium 또는 Puppeteer 같은 도구와 통합해야 합니다.

IP 차단 문제가 발생하거나 커스텀 ヘッダー가 필요한 경우, AutoScraper는 requests 모듈의 リクエスト에 대해 다음과 같이 추가 リクエスト パラメータ를 지정할 수 있습니다:

```python
# build the scraper on an initial URL
scraper.build(
    url,
    wanted_list=wanted_list,
    request_args=dict(proxies=proxies) # this is where you can pass in a list of proxies or customer headers
)
```

예를 들어, AutoScraper로 스크레이핑할 때 커스텀 user agent와 プロキシ를 설정하는 방법은 다음과 같습니다:

```python
request_args = { 
  "headers: {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_5) AppleWebKit/537.36 \
            (KHTML, like Gecko) Chrome/84.0.4147.135 Safari/537.36"  # You can customize this value with your desired user agent. This value is the default used by Autoscraper.
  },
  "proxies": {
    "http": "http://user:[email protected]:3128/" # Example proxy to show you how to use the proxy port, host, username, and password values
  }
}
# build the scraper on an initial URL
scraper.build(
    url,
    wanted_list=wanted_list,
    request_args=request_args
)
```

AutoScraper는 Python의 requests 라이브러리를 사용하며 レート制限을 지원하지 않습니다. レート制限을 처리하려면 수동으로 스로틀링(throttling)을 구현하거나, 미리 만들어진 솔루션인 [`ratelimit`](https://pypi.org/project/ratelimit/) 라이브러리를 사용할 수 있습니다.

AutoScraper는 비동적 웹사이트에서만 동작하므로 CAPTCHA로 보호된 사이트를 처리할 수 없습니다. 이러한 경우에는 LinkedIn, Amazon, Zillow와 같은 사이트에서 구조화된 데이터 추출을 지원하는 [Bright Data Web Scraping API](https://brightdata.co.kr/products/web-scraper)와 같은 보다 강력한 솔루션을 고려하십시오.

지금 가입하고 무료 체험을 포함한 Bright Data의 제품을 탐색해 보십시오!