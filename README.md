# KAMP 유압펌프 이상탐지 — PatchTST 마스킹 재구성

정상 데이터만으로 학습한 PatchTST(마스킹 재구성)로 유압펌프 이상 구간을 탐지하고, 단계별 개선 실험과 팀 통일 지표 재평가까지 담은 저장소입니다.

## 데이터
- `press_data_normal.csv` — 정상 20,000행 (2022-07-12 00:00~01:16)
- `outlier_data.csv` — 이상 600행 (2022-07-17 10:51~10:53)
- 컬럼: TimeStamp, AI0_Vibration, AI1_Vibration, AI2_Current, Equipment_state

## 노트북 (실행 순서)
| 파일 | 내용 | 결과 |
|---|---|---|
| `EDA_hydraulic_pump.ipynb` | 데이터 진단 (불균형 33:1, 버스트 수집 구조, 중복, 저진폭 구간, 교류 전류) | 전처리 방향 |
| `PatchTST_anomaly_detection.ipynb` | **최종본**. 전처리 → PatchTST → 임계값 → 베이스라인 비교 → 결과 분석 → 제출 파일 | PR-AUC 0.949, F1 0.909 |
| `PatchTST_stage1_window.ipynb` | 개선 1: WINDOW 16/24/32 비교 | 효과 없음, 미탐 원인 규명 |
| `PatchTST_stage2_padmask.ipynb` | 개선 2: 패딩 패치 attention 제외 (pad-aware) | 미탐 67→56, F1 0.919 |
| `PatchTST_stage6_threshold.ipynb` | 개선 6: 운전 조건·세그먼트 길이별 임계값 | 효과 없음, 임계값 편향 한계 발견 |
| `PatchTST_stage3_postprocess.ipynb` | 개선 3: 후처리 (갭 가중치 + 연속 판정) | F1 0.936, 거짓 경보 28→5회 |
| `PatchTST_eval_team_metrics.ipynb` | 팀 통일 지표(Low-sev Recall, Worst Mode FPR 등)로 단계별 재평가 | 비교표 |

각 노트북은 위에서 아래로 Run All 하면 전처리부터 결과 파일까지 자동 실행됩니다. 출력은 `patchtst_outputs*/` 폴더에 저장됩니다.

## 최종 결과 (팀 통일 지표)
| Model | F1 | PR-AUC | Recall | FPR | Low-sev Recall | Worst Mode FPR |
|---|---|---|---|---|---|---|
| PatchTST 기준선 | .909 | .949 | .888 | 1.00% | 86.9% | 3.15% |
| + pad-aware | .919 | .956 | .907 | 1.00% | 90.1% | 3.23% |
| + pad-aware + 후처리 P4 | **.936** | .955 | .902 | **0.38%** | **91.0%** | **1.24%** |
| + pad-aware + 후처리 P5 | .931 | **.966** | **.920** | 0.85% | 90.1% | 2.82% |

## 환경
- Python 3.10, PyTorch 2.x (CUDA), numpy / pandas / scikit-learn / matplotlib / seaborn
- 외부 패키지 설치 없이 표준 환경에서 실행 가능. seed 고정으로 재실행 시 동일 결과.
