# 프로젝트 명 : FeelNa("Movie")
 ![image](https://user-images.githubusercontent.com/63627273/88992370-3337dc80-d31e-11ea-8c13-a325c3e6fa10.png)  
 20200713~20200731

## 감성키워드 AI추천시스템 (구 Movie For You)
- 사용자의 단어 입력에 따른 영화를 추천해주는 인공지능 추천 시스템입니다.
- 소셜 리뷰 데이터를 기반으로 입력값과의 코사인유사도를 측정하여 영화를 추천합니다.
- 입력된 단어와 유사한 단어를 word2Vec을 사용해 출력
- 관람객의 리뷰 데이터 기반 영화를 추천
- 추천된 영화를 기반으로 또 다시 유사한 영화를 제공  

## 데이터 수집
-  영화진흥위원회 영화DB에 존재하는 영화 중 2001년 ~ 2020년(총 20년간) 제작된 국내외 영화 총 8034건을 기반으로 각 플랫폼의 리뷰를 수집하였습니다.
1. 네이버 영화 리뷰
   * 기존 영화목록이 아닌 연도별 목차에서 영화목록을 가져왔습니다. (목록과 다른 부분은 재검색)
   * HTML코드가 일정하지 않은 부분이 있어서 예외처리(2개) 했습니다.
   * 연도 당 영화 평균 600개정도의 데이터를 크롤링 했습니다, 약 4만개
2. 네이버 블로그 리뷰
   * 영화진흥위원회 영화 데이터 베이스를 기반으로 하여 관객수가 백만이상의 영화에 대한 블로그 리뷰를 가져왔습니다(기간 : 2006-03-26 ~ 2020-07-18, 정렬 : 정확도순)
   * Selenium, webdriver를 이용했고, 블로그 타이틀, URL을 타고 들어가 내용 크롤링
   * 영화당 리뷰 5개씩 크롤링 (크롤링이 금지된 블로그의 리뷰, 네이버 검색어 검열에 걸리는 영화 2개 제외), 약 8천개
3. 다음 영화 리뷰
   * 다음영화리뷰 사이트에서 영화리스트의 title을 검색하여, 각 영화에 맞는 리뷰를 가져왔습니다.  
   * 검색 시, 검색은 되지만, 영화리뷰가 존재하지않는 경우가 있어 try~except를 사용하여  NoSuchElementException을 예외처리하였습니다.     
   * 다음 영화사이트의 경우, 영화마다의 코드와 페이지 변경 방식이기 때문에(리뷰를 선택하면 새탭으로 창이 열림), selunium을 사용하여 페이지 변경을 반영하였습니다.     
   * 정적페이지의 리뷰를 받아오기 위해서, BS4를 이용해 10페이지까지의 리뷰를 수집하였습니다, 약 80만개
4. 왓챠피디아 영화 리뷰
   * 왓챠피디아 (구 왓챠)는 국내에서 제일 많이 쓰는 영화, TV 프로그램 추천 프로그램 앱입니다.  
   * 검색창에 영화 제목을 검색하여 접근하는 방식으로 크롤링을 진행. 리뷰가 스크롤을 내려야 로딩이 되는 방식이라 코드에 반영하였습니다. 
   * Beautifulsoup이 아닌 Selenium, webdriver 만을 이용해 태그로 접근하여 리뷰를 가져왔습니다. 
   * 모든 리뷰를 가져오는 것이 아니라서 스크롤 내리는 정도를 일정하게 지정하였습니다, 약 7만개
   
## 데이터정제
1. 크롤링 과정에서 분리되지 못한 태그 및 제어문자, 특수문자를 제거
2. 영어와 달리 한국어 불용어 목록은 아주 기본적인 것만 있음. 영화의 특성을 반영할 수 있는 데이터를 남기기 위해서 영화리뷰 전용 불용어 사전을 제작
3. 전체 문서에서 2번 미만 등장한 단어들을 제작된 불용어 사전에 추가
4. 정제된 데이터를 바탕으로 코퍼스 구성
    
## Word2Vec 리뷰키워드 기반 추천시스템 구현
1. 사용자가 키워드를 입력하면 (ex. 아련한) Word2vec 을 통해 단어를 출력. 
2. 출력된 단어 중 사용자가 원하는 키워드를 연속 선택
3. 선택된 단어들과 유사한 단어 리스트로 가상의 리뷰를 만들고, 기존의 리뷰 코퍼스와 함께 코사인 유사도를 진행
4. 사용자가 선택한 단어들과 관련된 한줄 리뷰와 영화목록을 출력.  

        from gensim.models import Word2Vec
        embedding_model = Word2Vec(sentences, size= 100, window = 4, min_count= 20, workers=4, iter =100, sg= 1)
        embedding_model.save('word2VecModel')

## Consine Similarity 유사영화 추천시스템 구현
1. 사용자가 선택한 영화를 영화 리뷰 코퍼스를 기반으로 코사인 유사도 사용.
2. 해당 영화와 유사한 영화를 유사도 순으로 정렬하여 출력

        # tf-idf 행렬만들기
        from sklearn.feature_extraction.text import TfidfVectorizer
        tfidf = TfidfVectorizer()
        tfidf_matrix = tfidf.fit_transform(df.review) 
        
        from sklearn.metrics.pairwise import linear_kernel
        cosine_sim = linear_kernel(tfidf_matrix, tfidf_matrix)

## 구현내용 요약
1. Movie4You 결과물 폴더 내 ipynb 노트북 파일 실행을 통해 구동할 수 있습니다. 
   * 0730추천시스템구현 파일은 일부 영화에 대해 마지막 시각화에 문제가 생기는 부분에 대해 우선 주석처리를 한 파일로 마지막 워드클라우드가 나타나지 않습니다. 
   * 함께 들어가있는 피클과 모델, csv 모두 구동에 필요한 파일입니다. 
2. 실행 후 입력창에 첫 번째 키워드를 입력합니다. 
3. 출력된 단어들과 시각화를 확인 후 원하는 키워드를 선택해 다시 입력창에 입력합니다. 
4. 같은 방식으로 세 번째 키워드도 입력합니다. 
5. 출력된 리뷰와 영화목록을 확인할 수 있습니다. 
6. 출력된 목록 중에서 원하는 영화를 다시 입력하면 해당 영화와 유사한 영화목록이 유사도 순으로 출력됩니다. 
