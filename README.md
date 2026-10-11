# 사출성형 AI 파이프라인 — 공모전 제출 안내

원본 구조의 `molding_ai_repro/` 소스코드 폴더와 `04. Dataset_Molding.zip`은 유지하고, 설명 문서와 통합 테스트 예측결과를 바깥 제출 폴더로 배치했습니다. Markdown 설명 문서는 `README.md` 하나로 통합했습니다.

`원본 데이터 → Shot 단위 통합 / 30개 피처 → kNN 상위 10% 선별 → 재정규화 → SCARF 사전학습 → MLP/FT-Transformer 학습·평가 → 비교 보고서 및 테스트 예측결과`

## 1. 제출 폴더 및 파일 역할

압축을 풀면 `KAMP 공모전_JOHAN` 폴더에 다음 네 가지 항목이 함께 있습니다.

| 최상위 항목 | 역할 |
|---|---|
| `molding_ai_repro/` | 소스코드, 의존성 파일, 상세 실험 결과, 비교 기준 및 코드 확인용 파일 |
| `04. Dataset_Molding.zip` | 학습·전처리에 사용하는 원본 데이터. 압축을 풀지 않고 코드가 내부 CSV를 읽음 |
| `README.md` | 실행 방법, 전체 파일·폴더 역할, 테스트 예측결과, 재현 조건 및 검증 내역 |
| `prediction_output.csv` | 공모전 제출용 통합 테스트 예측결과 |

전체 ZIP은 데이터와 원본의 저장 모델·전처리 중간 파일을 포함합니다. 다운로드용 경량 ZIP에는 코드와 개별 예측 CSV·성능 지표를 넣었으며, 데이터 ZIP·저장 모델·큰 전처리 파일은 제외했습니다. 경량 ZIP을 사용하면 기존 `04. Dataset_Molding.zip`을 `README.md`와 같은 바깥 폴더에 복사합니다.

### `molding_ai_repro/` 내부 파일과 폴더

| 항목 | 역할 |
|---|---|
| `run_all.py` | 전처리 → 사전학습 → 모델 학습·평가 → 비교 보고서 → 바깥 `prediction_output.csv` 생성을 실행하는 메인 코드 |
| `export_test_predictions.py` | 저장된 개별 예측 CSV를 바깥 `prediction_output.csv`로 통합. 학습은 수행하지 않음 |
| `requirements.txt` | 필요한 Python 패키지. 기본 설치에 사용 |
| `requirements-reference.txt` | 과거 노트북의 XGBoost·LightGBM·CatBoost 고정 버전 기록. 기본 설치에는 자동으로 포함되지 않음 |
| `run_pipeline.bat` | Windows에서 `run_all.py`를 실행하는 보조 파일 |
| `run_pipeline.sh` | macOS/Linux에서 `run_all.py`를 실행하는 보조 파일 |
| `src/` | 실제 전처리·모델·학습·보고서 코드 |
| `outputs/full_run/` | 전처리 파일, 개별 예측 CSV, 성능 지표, 학습 설정, 비교 그림 및 저장 모델이 생성되는 폴더 |
| `reference/` | 과거 CN7 결과와 비교하는 기준 CSV. 학습 입력이나 예측값 생성에 사용하지 않음 |
| `tests/smoke_test.py` | 30개 피처 및 두 인코더·SCARF의 기본 작동을 확인하는 코드 |
| `.venv/` | 사용자가 생성하는 Python 가상환경. 배포 ZIP에는 포함하지 않음 |

### `src/` 내부 코드

| 파일 | 역할 |
|---|---|
| `src/constants.py` | 시드, 손실함수, 모델 목록, 기존 실험 점검 기준 등 공통 설정 |
| `src/data.py` | 데이터 ZIP에서 CSV 읽기, 제품별 데이터 선택, 검증된 LH/RH 쌍의 Shot 단위 통합 |
| `src/preprocess.py` | 피처 생성·정규화, kNN 기반 비라벨 데이터 선별, 재정규화, 학습·검증·테스트 및 SSL 입력 구성 |
| `src/models.py` | MLP/FT-Transformer 인코더, SCARF, 분류 모델 및 모델 학습·평가 함수 |
| `src/train.py` | 학습 데이터에만 SMOTE/ADASYN 적용, 시드·손실함수별 실험 실행, 개별 모델·지표·예측 저장 |
| `src/report.py` | 성능 평균·표준편차 집계, 인코더 비교 히트맵 및 재현 점검 보고서 생성 |
| `src/reference.py` | 과거 CN7 비교 수치를 기준 CSV로 생성하는 코드 |
| `src/__init__.py` | `src` 폴더를 Python 패키지로 구성하는 파일 |

## 2. 설치 및 실행 방법

Python 3.11 이상을 권장합니다. 아래 Windows PowerShell 명령은 `README.md`가 있는 바깥 `KAMP 공모전_JOHAN` 폴더에서 실행합니다.

```powershell
python -m venv .\molding_ai_repro\.venv
.\molding_ai_repro\.venv\Scripts\python.exe -m pip install -r .\molding_ai_repro\requirements.txt
.\molding_ai_repro\.venv\Scripts\python.exe .\molding_ai_repro\run_all.py
```

첫 줄은 가상환경을 생성합니다. 두 번째 줄은 해당 가상환경 안에 패키지를 설치합니다. 세 번째 줄은 가상환경의 Python으로 전체 파이프라인을 실행합니다. 별도 활성화 명령은 필요하지 않습니다.

macOS/Linux에서는 바깥 폴더에서 다음 명령을 실행합니다.
기본 위치를 KAMP 공모전_JOHAN에 놓은 뒤 아래 명령어를 차례대로 실행합니다.

```bash
python -m venv molding_ai_repro/.venv
.\molding_ai_repro\.venv\Scripts\activate
python -m pip install -r molding_ai_repro/requirements.txt
python molding_ai_repro/run_all.py
```

기본 데이터 경로는 소스코드 폴더의 상위 폴더에 있는 `04. Dataset_Molding.zip`입니다. 기본 실행은 CN7/RG3, MLP/FT-Transformer, 시드 42/43/44, BCE/Cost/Focal/Cost+Focal 조건을 사용합니다.

Windows에서 빠른 연결 확인:

```powershell
.\molding_ai_repro\.venv\Scripts\python.exe .\molding_ai_repro\run_all.py --quick --products cn7 --no-save-models
```

전처리만 실행:

```powershell
.\molding_ai_repro\.venv\Scripts\python.exe .\molding_ai_repro\run_all.py --preprocess-only
```

학습 없이 기존 예측결과만 다시 통합:

```powershell
.\molding_ai_repro\.venv\Scripts\python.exe .\molding_ai_repro\export_test_predictions.py
```

다음 과거 입력 사용 모드 예시는 `molding_ai_repro` 폴더에서 필요한 패키지가 설치된 Python 환경으로 실행합니다.

## 3. 테스트 예측결과와 상세 출력

바깥 `prediction_output.csv`에는 원본의 실제 예측 CSV 112개 실험, 총 13,664행을 통합했습니다. 점수와 예측값은 원본과 같습니다.

- CN7 / MLP: 시드 42/43/44 × 4개 손실함수 × 8개 모델 = 96개 실험.
- CN7 / FT-Transformer: 시드 42 × BCE/Cost × 8개 모델 = 16개 실험.
- 실험당 122개 테스트 Shot이며, 동일 Shot이 실험별로 반복됩니다.
- 원본에 없는 RG3 예측결과 및 나머지 FT-Transformer 실험은 현재 CSV에 포함하지 않았습니다. 기본 전체 실행 범위와 기존 저장 결과의 범위는 다릅니다.

| CSV 열 | 의미 |
|---|---|
| `product`, `encoder` | 제품과 인코더 |
| `loss`, `model`, `path`, `seed` | 손실함수, 분류 모델, 학습 경로, 난수 시드 |
| `threshold` | 해당 실험의 예측 임계값 |
| `shot_id` | 테스트 Shot 식별자 |
| `y_true` | 실제 라벨: 0=양품, 1=불량 |
| `score` | 모델 추론 점수. SVM 등은 확률이 아닌 점수일 수 있음 |
| `prediction` | 예측 라벨: 0=양품, 1=불량 |
| `source_file` | `molding_ai_repro/outputs/full_run/` 기준 원본 예측 CSV 경로 |

학습·평가가 정상 종료되면 바깥 `prediction_output.csv`를 자동 갱신합니다. 전처리만 실행하면 예측결과는 생성하지 않습니다. 통합 시 각 인코더 폴더의 `metrics.csv`에 기록되어 있고 대응하는 예측 CSV가 존재하는 실험을 사용합니다. 결과 폴더를 재사용하면 남아 있는 다른 제품·인코더 결과도 함께 포함될 수 있습니다. 별도 결과 폴더는 `--output`으로 지정할 수 있습니다.

| `molding_ai_repro/outputs/full_run/` 내부 경로 | 역할 |
|---|---|
| `<product>/<encoder>/metrics.csv` | 모델·손실함수·시드별 성능과 임계값 |
| `<product>/<encoder>/*_test_predictions.csv` | 한 실험의 테스트 Shot별 실제 라벨·점수·예측값 |
| `all_metrics.csv` | 해당 실행에서 수집한 평가 결과 |
| `summary_mean_sd.csv` | 해당 실행에서 집계한 평균·표준편차 |
| `training_config.json` | 해당 실행의 제품·인코더·시드·손실함수 등 설정 |
| `reproduction_report.json` | 앞단 데이터 개수와 과거 성능 기준의 비교 |
| `preprocessing/` | 선별 데이터·재정규화 CSV·SSL 배열 등 전처리 산출물 |
| `*_encoder_comparison.png` | 제품별 인코더 성능 비교 히트맵 |
| `*.pt`, `*.joblib` | 모델 저장 옵션 사용 시 생성되는 모델·변환기 |

기존 상세 결과는 원본의 값을 유지했습니다. 통합 CSV는 각 인코더의 개별 예측 CSV와 `metrics.csv`를 기준으로 만들었습니다. 기존 통합 지표나 그림을 다시 학습한 결과로 바꾸지 않았습니다.

## 4. 기존 결과를 정확하게 재현하기 위한 조건

업로드된 MLP 노트북은 과거 실험의 SCARF 입력을 `04. Dataset_Molding.zip`만으로 생성하지 않았습니다. 다음 세 가지 외부 파일을 명시적으로 불러왔습니다.

- `extract_top10_to_excel.py`에서 사용한 `CN7_RG3_shot_preprocessed_splits.xlsx` 또는 `_2.xlsx` 파일.
- 위 워크북을 바탕으로 생성한 `CN7_RG3_unlabeled_top10_indomain.xlsx`.
- 재정규화 스크립트에서 생성한 `moldset_{cn7,rg3}_unlabeled_final_scarf_ready.csv`.

첫 번째 워크북은 업로드된 파일에 포함되어 있지 않았습니다. 따라서 이 패키지는 두 가지 모드를 제공합니다.

1. **기존 입력 사용 모드**: 해당 워크북을 준비한 뒤 다음 명령어를 실행합니다.

   ```bash
   python run_all.py --split-workbook /path/to/CN7_RG3_shot_preprocessed_splits.xlsx \
     --selection-source-dir /path/to/original_preprocessing_folder
   ```

   상위 10% 선별과 재정규화 단계는 해당 기준 데이터를 대상으로 업로드된 전처리 로직을 그대로 적용합니다. 지정한 폴더에 `moldset_unlabeled_<product>_2.csv`가 있으면 원본 `extract_top10_to_excel.py`와 동일하게 이 파일을 우선 사용합니다. 없으면 일반 CSV를 사용합니다.

2. **환경 독립적 재구성 모드(기본값)**: `Dataset_Molding.zip`에서 Shot 단위 학습 기준 데이터를 결정론적으로 재구성합니다. 여러 환경에서 전체 파이프라인을 실행할 수 있지만, 누락된 과거 워크북과 바이트 단위로 동일한 데이터를 생성한다고 보장하지는 않습니다.

`reproduction_report.json`은 기존 노트북에 기록된 앞단 전처리 데이터 개수를 확인합니다. CN7은 선별 3524행 → SSL 학습/검증 3170/353행이며, RG3는 선별 3596행에서 겹치는 데이터 200행과 중복 데이터 1677행을 제거한 뒤 SSL 학습/검증 1547/172행입니다. 이 개수가 일치하지 않으면 최종 성능 차이를 모델 변경의 영향으로 해석하기 전에, 앞단 전처리 데이터의 차이부터 확인해야 합니다.

## 5. 성능 비교표의 각 셀이 의미하는 것

각 인코더와 네 가지 손실함수 열에 대해 먼저 라벨 없이 SCARF 사전학습을 수행합니다. 이후 해당 열의 손실함수로 인코더와 MLP 헤드를 함께 미세조정합니다. `MLP joint`는 이렇게 직접 학습한 모델입니다. 후속 모델인 SVM/XGBoost/LightGBM/CatBoost/SingleCNN/ResidualMLP는 이 과정에서 얻은 256차원 인코더 표현을 입력으로 학습합니다. `ResidualMLP joint`는 동일한 사전학습 인코더를 잔차 헤드와 함께 미세조정합니다.

손실함수 열은 **인코더와 헤드의 공동 미세조정에 사용하는 손실함수**를 뜻합니다. 업로드된 노트북과 동일하게, 후속 SVM/부스팅 모델은 비용 가중치를 유지하고 후속 신경망 모델은 내부적으로 Cost+Focal을 사용해 학습합니다.

## 6. 노트북에서 유지한 재현 설정

- 시드: 42, 43, 44.
- SCARF 입력 교란: 30개 열의 정확히 60%인 18개 피처를 데이터에서 관측된 피처별 주변분포의 다른 샘플 값으로 대체.
- 인코더 출력: 256차원.
- MLP 인코더: Linear+ReLU로 구성된 4개 층.
- FT-Transformer: 수치형 피처 토큰 30개와 CLS 토큰, 토큰 차원 64, 블록 2개, 헤드 4개, ReGLU 방식의 FFN, 256차원 출력 어댑터.
- 공동 학습 최적화: Adam, 학습률 1e-3, 검증 AP 기준 체크포인트 선택, 조기 종료 patience 3.
- 후속 신경망 모델: AdamW, 학습률 1e-3, 가중치 감쇠 1e-4, 100에포크.
- 검증 임계값: 양품의 오탐률(FPR)이 5% 이하라는 조건을 만족하는 가장 큰 임계값.
- 테스트 데이터는 증강 전에 분리하며, SMOTE/ADASYN은 학습 데이터에만 적용.

제공된 과거 비교 그림에 표시된 반올림된 CN7 수치는 `molding_ai_repro/reference/cn7_historical_reference.csv`에 검증 기준으로만 저장되어 있습니다. 이 수치를 예측 생성에 사용하거나 학습 결과를 덮어쓰는 데 사용하지 않습니다.


## 7. 검증 내역과 범위

- 원본 예측결과 112개 실험, 13,664행을 통합했습니다.
- Shot 식별자·실제 라벨·점수·예측값이 원본과 동일함을 확인했습니다.
- 예측값과 실험별 임계값이 일치함을 확인했습니다.
- Python 파일의 문법과 결과 통합 명령 실행을 확인했습니다.
- ZIP 압축 무결성을 확인했습니다.
- 전체 ZIP은 원본 데이터 ZIP·학습 모델·전처리 중간 파일을 포함합니다. 경량 ZIP은 이 큰 파일들을 제외합니다.
- 경량 ZIP을 사용하는 경우 기존 데이터 ZIP을 최상위 폴더에 추가해야 합니다.
- 전체 모델 학습을 다시 수행하지 않았습니다.

이번 문서 통합에서는 기존 네 개 Markdown 문서의 설명을 이 README에 모았습니다. 실행 코드와 데이터 예측 CSV는 변경하지 않았습니다. 최종 ZIP에는 Markdown 문서가 `README.md` 하나만 있습니다.

## 8. 과거 기준 자료의 출처와 사용 범위

`molding_ai_repro/reference/cn7_historical_reference.csv`는 제공된 MLP·FT-Transformer 노트북 및 CN7 비교 그림에 표시된 반올림된 평균과 표준편차를 담은 검증 기준입니다. 계산된 지표를 이 수치로 바꾸거나 대체하지 않습니다.

제공된 원본 코드에서 과거 입력을 정확히 재현하려면 외부 `CN7_RG3_shot_preprocessed_splits*.xlsx` 워크북이 필요했지만, 해당 파일은 제공된 자료에 없었습니다. 따라서 모델 수준의 성능 차이를 해석하기 전에 재구성한 앞단 전처리 데이터의 개수가 기존 노트북 점검 기준과 일치하는지 보고합니다.
