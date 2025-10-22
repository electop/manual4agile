## Procedure
* Agent Builder 접속
  * AI Playground 접속 > AI 서비스 및 도구 모음 > Agent Builder 접속
* Agent Template 선택
  * 화면 좌측 상단 My Projects > + New Agent
  * 화면 좌측 상단 Templates > All templates > Simple Chat Agent
* 추론 모델 선택
  * 작업창에서 Component (Gauss2.3) 삭제
  * 화면 좌측 상단 Components > Model > Component (GaussO Flash) + 선택
    * Component (GaussO Flash)의 "< > Edit Parameters" 선택
      * temperature 값을 0.3에서 0.1로 수정 > Save
* 데이터 입력
  * 화면 좌측 상단 Components > Input / Output > Component (File +) 선택
  * Component (File) > Select files > Click or drag files here > "projects_by_id_marked_v2.md" 파일 Drag & Drop
* Wiring
  * 연결 #01 : Component (File)의 output (우측 아래) ↔ Component (GaussO Flash)의 input (좌측 위)
  * 연결 #02 : Component (Chat)의 output (우측 아래) ↔ Component (GaussO Flash)의 input (좌측 아래)
  * 연결 #03 : Component (GaussO Flash)의 output (우측 아래) ↔ Component (Chat Output)의 input (좌측 중)
* 실행
  * 화면 우측 상단 아이콘 메뉴 > "Run with Chat" 실행
