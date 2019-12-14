# 기록

## 2019.12.11

- pandas 에서 대용량 csv 를 읽어서 처리하는 작업
- csv 를 직접 access 할 경우, Jupyter notebook kernel 이 자동으로 restart 되는 문제가 간혹 발생
- 작업 시작 후 10초 이상 경과하는 경우에 그와 같은 문제가 발생하는 것으로 추정됨
- googling 결과 Jupyter notebook 을 쓰지 말라는 답변 보임
- read_csv 로 csv 파일을 직접 처리하면 시간이 오래 걸리기 때문에 hdf5 나 pickle(pkl) 사용하면 문제 해결 가능
- pkl 보다 범용성이 좋은 hdf5 를 사용하기로 함
- read 성능만 보면 pkl 이 더 빠르다고 함
- pandas 에 read_hdf 기능 있음
- dask 를 사용하는 방법도 있음

----

## 2019.12.12

### convert csv to feather
- googling 중 feather format 의 성능이 더 좋다는 내용을 보고 feather format 으로 변환하여 사용함([link](https://towardsdatascience.com/the-best-format-to-save-pandas-data-414dca023e0d))

----

## machine learning

[machine learning](2019-12-12-machine learning.md)
