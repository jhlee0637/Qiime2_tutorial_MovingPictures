# Overview
이 튜토리얼은 [Qiime2의 "Moving Pictures" tutorial](https://docs.qiime2.org/2023.9/tutorials/moving-pictures/)을 참고하였습니다.

이 튜토리얼의 목적은 작업환경에서 분석까지 스스로 구성해서 수행하는 것입니다.

따라서 이 튜토리얼은 다음을 수행합니다.
- Windows11에 Docker Desktop 설치 및 WSL2와 Docker 연결
- Qiime2를 비롯한 metagenomics pipeline 구성
- Metagenomics 데이터 분석(taxonomical, phylogenetical)

이 튜토리얼을 더 잘 이해하기 위해서는 Bash 프로그래밍 언어를 이해하실 필요가 있습니다.
# 1. Win11에 Docker Desktop 설치 및 WSL2와 연결
- [windows 11에 docker 설치하기 (wsl2 이슈 해결) 2022. 3. 26.](https://herojoon-dev.tistory.com/120)
- https://docs.docker.com/desktop/wsl/#turn-on-docker-desktop-wsl-2
- 설치 완료 후 다음의 명령어를 입력하여 WSL2와 Docker 사이의 연결 확인
	- WSL2를 사용할 환경은 편하신대로 VS code, WSL, git bash 등의 프로그램을 활용하면 됩니다.
```bash
$ docker --version

Docker version 24.0.2, build cb74dfc
```
# 2. Docker를 활용하여 Qiime2 image 다운로드 및 container 설치
```shell
$ docker pull quay.io/qiime2/core
```

```bash
mkdir qiime2-moving-pictures-tutorial
cd qiime2-moving-pictures-tutorial
```
- Docker Container를 실행하기 전에 먼저 연습하려는 폴더로 이동해주세요.

```shell
$ docker run --name "qiime2" -t -i -d -v $(pwd):/data quay.io/qiime2/core
```
- Qiime2 홈페이지 설명과는 다르게 코드가 적혀있습니다.
	- 끝의 command qiime을 제거하였고, 날짜와 상관없이 최신버전을 받도록 하였습니다.
- -v 옵션에 의해서 현재 위치한 폴더가 qiime2 안의 /data 경로에 mount됩니다.
- Container의 이름은 "qiime2"로 지정하였으나, 다른 이름으로 수정하여도 되고, 혹은 아예 --name 옵션을 생략하여도 다른 무작위 이름으로 container가 생성됩니다.
- Run command를 사용한 순간부터 container가 실행 중인 상태여야 합니다. 
	- `docker ps`로 실행 중인 container를 확인해주세요.
## Q. Docker가 아닌 Conda로 Qiime을 설치하고 싶다면?
- Qiime2 메뉴얼에서는 실제로 Conda로 설치하는 방법도 소개하고 있습니다.
	- Docker와 Conda는 프로그램을 설치한다는 점에서는 비슷하게 보일 수 있지만, Conda는 파이썬 레벨에서 작업환경을 나누는 것이고 Docker는 OS 레벨에서 작업환경을 나눈다는 점에서 차이점이 있습니다.
- Docker로는 Qiime2뿐 아니라 Linux system을 통째로 구성할 수 있지만, Conda는 Linux system과 같은 OS를 다루지 못합니다.
- 실제로 Conda를 사용해서 Qiime2를 설치하는 것도 방법이지만, 산업계에서 애용하는 Docker와 친숙해지는 것을 개인적으로 추천합니다.
# 3. Qiime2를 연습할 파일 다운로드
## 연습에 사용할 Metadata의 간략한 정보 다운로드
```bash
wget -O "sample-metadata.tsv" "https://data.qiime2.org/2023.9/tutorials/moving-pictures/sample_metadata.tsv"
```
## Metadata 다운로드
```bash
mkdir emp-single-end-sequences
```
### Barcord Sequence FASTQ 파일 다운로드
```bash
wget -O "emp-single-end-sequences/barcodes.fastq.gz" "https://data.qiime2.org/2023.9/tutorials/moving-pictures/emp-single-end-sequences/barcodes.fastq.gz"
```
### Sequence FASTQ 파일 다운로드
```bash
wget -O "emp-single-end-sequences/sequences.fastq.gz" "https://data.qiime2.org/2023.9/tutorials/moving-pictures/emp-single-end-sequences/sequences.fastq.gz"
```
### Qiime2 프로세스 접근 및 확인
```shell
$ docker exec -t -i qiime2 bash
```
- Container의 이름을 'qiime2'로 지정하였기 때문에 qiime2라고 적었습니다.
- CONTAINER_ID로 접근할 수도 있습니다.

```bash
# ls

emp-single-end-sequences  sample-metadata.tsv
```
- 볼륨이 제대로 mount 되었는지 `ls` 명령어로 확인합니다.
### Import EMPSingleEndSequences Tools
`EMPSingleEndSequences`는 Qiime2 multiplxed sequence 중 paired-end가 아닌 single-end로 이루어진 데이터를 다룬다.
```bash
qiime tools import --type EMPSingleEndSequences --input-path emp-single-end-sequences --output-path emp-single-end-sequences.qza
```
# 4. 분석
## Demultiplexing Sequence
```bash
qiime demux emp-single --i-seqs emp-single-end-sequences.qza --m-barcodes-file sample-metadata.tsv --m-barcodes-column barcode-sequence --o-per-sample-sequences demux.qza --o-error-correction-details demux-details.qza
```
- `emp-single`: 주어지는 input data가 single-end reads

Demultiplexing Summary 파일을 작성하는 것도 좋다.
```bash
qiime demux summarize --i-data demux.qza --o-visualization demux.qzv
```
- `--i-date`: input data
- `--o-visualization`: output data
	- https://view.qiime2.org/ 페이지에서 볼 수 있다.
## Sequence Quality Control by DADA2
- DADA2 이외에도 [Deblur](https://docs.qiime2.org/2023.9/tutorials/moving-pictures/#option-2-deblur)를 사용하여 sequene 데이터에 대한 QC가 가능하다.
```bash
qiime dada2 denoise-single --i-demultiplexed-seqs demux.qza --p-trim-left 0 --p-trunc-len 120 --o-representative-sequences rep-seqs-dada2.qza --o-table table-dada2.qza --o-denoising-stats stats-dada2.qza
```

QC결과를 시각화할 수도 있다.
```bash
qiime metadata tabulate --m-input-file stats-dada2.qza --o-visualization stats-dada2.qzv
```
## FeatureTable and FeatureData summaries
```bash
mv rep-seqs-dada2.qza rep-seqs.qza
mv table-dada2.qza table.qza
```
- DADA2의 output을 featuretable로 사용할 것이다.
```shell
qiime feature-table summarize --i-table table.qza --o-visualization table.qzv --m-sample-metadata-file sample-metadata.tsv
```

```shell
qiime feature-table tabulate-seqs --i-data rep-seqs.qza --o-visualization rep-seqs.qzv
```
## Generate a tree for phylogenetic diversity analyses
```shell
qiime phylogeny align-to-tree-mafft-fasttree --i-sequences rep-seqs.qza --o-alignment aligned-rep-seqs.qza --o-masked-alignment masked-aligned-rep-seqs.qza --o-tree unrooted-tree.qza --o-rooted-tree rooted-tree.qza
```
## Alpha and beta diversity analysis
- Alpha diversity
- Beta diversity
```shell
qiime diversity core-metrics-phylogenetic \
  --i-phylogeny rooted-tree.qza \
  --i-table table.qza \
  --p-sampling-depth 1103 \
  --m-metadata-file sample-metadata.tsv \
  --output-dir core-metrics-results
```
### Alpha diversity
```shell
qiime diversity alpha-group-significance \
  --i-alpha-diversity core-metrics-results/faith_pd_vector.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization core-metrics-results/faith-pd-group-significance.qzv

qiime diversity alpha-group-significance \
  --i-alpha-diversity core-metrics-results/evenness_vector.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization core-metrics-results/evenness-group-significance.qzv
```
### Beta diversity
```shell
qiime diversity beta-group-significance \
  --i-distance-matrix core-metrics-results/unweighted_unifrac_distance_matrix.qza \
  --m-metadata-file sample-metadata.tsv \
  --m-metadata-column body-site \
  --o-visualization core-metrics-results/unweighted-unifrac-body-site-significance.qzv \
  --p-pairwise

qiime diversity beta-group-significance \
  --i-distance-matrix core-metrics-results/unweighted_unifrac_distance_matrix.qza \
  --m-metadata-file sample-metadata.tsv \
  --m-metadata-column subject \
  --o-visualization core-metrics-results/unweighted-unifrac-subject-group-significance.qzv \
  --p-pairwise
```
### Visualization
```shell
qiime emperor plot \
  --i-pcoa core-metrics-results/unweighted_unifrac_pcoa_results.qza \
  --m-metadata-file sample-metadata.tsv \
  --p-custom-axes days-since-experiment-start \
  --o-visualization core-metrics-results/unweighted-unifrac-emperor-days-since-experiment-start.qzv

qiime emperor plot \
  --i-pcoa core-metrics-results/bray_curtis_pcoa_results.qza \
  --m-metadata-file sample-metadata.tsv \
  --p-custom-axes days-since-experiment-start \
  --o-visualization core-metrics-results/bray-curtis-emperor-days-since-experiment-start.qzv
```
## Alpha rarefaction plotting
```shell
qiime diversity alpha-rarefaction \
  --i-table table.qza \
  --i-phylogeny rooted-tree.qza \
  --p-max-depth 4000 \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization alpha-rarefaction.qzv
```
## Taxonomic analysis
### download data
```shell
wget \
  -O "gg-13-8-99-515-806-nb-classifier.qza" \
  "https://data.qiime2.org/2023.9/common/gg-13-8-99-515-806-nb-classifier.qza"
```
### Analysis
```shell
qiime feature-classifier classify-sklearn \
  --i-classifier gg-13-8-99-515-806-nb-classifier.qza \
  --i-reads rep-seqs.qza \
  --o-classification taxonomy.qza

qiime metadata tabulate \
  --m-input-file taxonomy.qza \
  --o-visualization taxonomy.qzv
```
### Visualization
```shell
qiime taxa barplot \
  --i-table table.qza \
  --i-taxonomy taxonomy.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization taxa-bar-plots.qzv
```
## Differential abundance testing with ANCOM
```shell
qiime feature-table filter-samples \
  --i-table table.qza \
  --m-metadata-file sample-metadata.tsv \
  --p-where "[body-site]='gut'" \
  --o-filtered-table gut-table.qza
```

```shell
qiime composition add-pseudocount \
  --i-table gut-table.qza \
  --o-composition-table comp-gut-table.qza
```

```shell
qiime composition ancom \
  --i-table comp-gut-table.qza \
  --m-metadata-file sample-metadata.tsv \
  --m-metadata-column subject \
  --o-visualization ancom-subject.qzv
```

```shell
qiime taxa collapse \
  --i-table gut-table.qza \
  --i-taxonomy taxonomy.qza \
  --p-level 6 \
  --o-collapsed-table gut-table-l6.qza

qiime composition add-pseudocount \
  --i-table gut-table-l6.qza \
  --o-composition-table comp-gut-table-l6.qza

qiime composition ancom \
  --i-table comp-gut-table-l6.qza \
  --m-metadata-file sample-metadata.tsv \
  --m-metadata-column subject \
  --o-visualization l6-ancom-subject.qzv
```
# 참고
- https://www.youtube.com/watch?v=TQ58fmBq8oE
- https://www.lainyzine.com/ko/article/docker-exec-executing-command-to-running-container/