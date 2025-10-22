```
연도별 사업부별 참가자수를 집계해줘.
- 연도는 '과제 시작일'의 연도
- 사업부는 '과제 사업부'
- 괄호안의 숫자는 중복 참여자수
- 중복 참여자수는 2회 이상 참여하는 케이스만 집계
- 참가자수의 총합은 879명 (예)
| 연도 | BDC | 전장 | MX | DA | 네트워크 | VD | SR | 의료기기 | DPC | GCS | 전사 | 합계 |
|------|-----|------|----|----|----------|----|----|----------|-----|-----|------|------|
| 2021 | 1(0) | 1(0) | 3(0) | 0 | 3(1) | 0 | 0 | 0 | 0 | 0 | 0 | 8(1) |
| 2022 | 1(0) | 1(0) | 3(0) | 0 | 3(1) | 0 | 0 | 0 | 0 | 0 | 0 | 8(1) |
| 합계 | 2(0) | 2(0) | 6(0) | 0 | 6(2) | 0 | 0 | 0 | 0 | 0 | 0 | 16(2) |
```
```
모든 연도별, 모든 사업부별 참가자수를 집계해서 테이블로 보여줘.
- 연도는 '과제 시작일'의 연도가 기준
- 첨부된 모든 문서의 데이터를 메모리에 로드
- 참가자수는 Knox ID 기준으로 집계
- 참가자수는 집계수(중복참여자수)로 표기 (예) 6(2)
- 중복 참여자수는 2번 이상 참가하는 케이스만 집계
- 결과값은 템플릿을 따라서 마크다운 코드로 표기
| 연도 | BDC | 전장 | MX | DA | 네트워크 | VD | SR | 의료기기 | DPC | GCS | 전사 | 합계 |  
|------|-----|------|----|----|----------|----|----|----------|-----|-----|------|------|
| 2021 | 1(0) | 1(0) | 3(0) | 0 | 3(1) | 0 | 0 | 0 | 0 | 0 | 0 | 8(1) |  
| 2022 | 1(0) | 1(0) | 3(0) | 0 | 3(1) | 0 | 0 | 0 | 0 | 0 | 0 | 8(1) |  
| 합계 | 2(0) | 2(0) | 6(0) | 0 | 6(2) | 0 | 0 | 0 | 0 | 0 | 0 | 16(2) |
```
```
import json  
import os  

def json_to_kv_code(json_data):  
    """JSON 데이터를 Markdown KV 형식의 코드 블록으로 변환"""  
    def _format(value):  
        """값을 재귀적으로 포맷팅"""  
        if isinstance(value, dict):  
            return "\n".join(f"{k}: {_format(v)}" for k,v in value.items())  
          
        elif isinstance(value, list):  
            return "\n".join(f"- {_format(item)}" for item in value)  
          
        else:  
            return str(value).strip()  
      
    try:  
        parsed_data = json.loads(json_data)  
    except json.JSONDecodeError as e:  
        return f"```kv\nJSON 파싱 오류 발생:\n{e}\n```"  
      
    kv_content = _format(parsed_data)  
      
    return f"```kv\n{kv_content}\n```"  

def read_json_file(file_path):  
    """파일에서 JSON 데이터 읽기"""  
    try:  
        with open(file_path, 'r', encoding='utf-8') as f:  
            return f.read()  
    except Exception as e:  
        return f"파일 읽기 오류: {e}"  

def write_to_md_file(output, file_path):  
    """Markdown 내용을 파일에 저장"""  
    try:  
        with open(file_path, 'w', encoding='utf-8') as f:  
            f.write(output)  
        return True  
    except Exception as e:  
        return f"파일 저장 오류: {e}"  

def split_data_by_ids(data, ids, chunk_size=20):  
    """과제 ID별로 데이터 분할"""  
    # 과제 ID 추출 및 정렬  
    id_list = sorted(ids)  
      
    # 청크 분할  
    chunks = []  
    for i in range(0, len(id_list), chunk_size):  
        chunk_ids = id_list[i:i+chunk_size]  
        chunks.append(chunk_ids)  
      
    return chunks  

def get_chunk_filename(start_id, end_id):  
    """청크 파일명 생성 (SWCOOOO-SWCOOOO 형식)"""  
    return f"{start_id}-{end_id}.md"  

def extract_common_header(data):  
    """공통 헤더 추출 (과제 ID와 --- 항목)"""  
    common_keys = ['과제 ID', '---']  
    common_data = {}  
      
    for key in common_keys:  
        if key in data:  
            common_data[key] = data[key]  
      
    return common_data  

if __name__ == "__main__":  
    input_file = "projects_by_id.json"  
    output_dir = "output"  
      
    # 디렉토리 생성  
    os.makedirs(output_dir, exist_ok=True)  
      
    # JSON 파일 읽기  
    json_content = read_json_file(input_file)  
      
    # 오류 처리  
    if isinstance(json_content, str) and json_content.startswith("파일"):  
        print(json_content)  
        exit(1)  
      
    # JSON 파싱  
    data = json.loads(json_content)  
      
    # 공통 헤더 추출  
    common_header = extract_common_header(data)  
      
    # 공통 헤더를 Markdown 형식으로 변환  
    header_output = json_to_kv_code(json.dumps(common_header))  
      
    # 과제 ID 추출 (공통 헤더 제외)  
    project_ids = [k for k in data.keys() if k not in ['과제 ID', '---']]  
      
    # 20개 단위로 분할  
    chunks = split_data_by_ids(data, project_ids, chunk_size=20)  
      
    # 각 청크 처리  
    for i, chunk in enumerate(chunks):  
        # 청크 데이터 결합  
        chunk_data = {pid: data[pid] for pid in chunk}  
          
        # Markdown 변환  
        chunk_output = json_to_kv_code(json.dumps(chunk_data))  
          
        # 전체 내용 구성: 공통 헤더 + 청크 데이터  
        full_output = f"{header_output}\n\n{chunk_output}"  
          
        # 파일명 생성 (첫번째 ID와 마지막 ID 사용)  
        start_id = chunk[0]  
        end_id = chunk[-1]  
        filename = get_chunk_filename(start_id, end_id)  
        output_path = os.path.join(output_dir, filename)  
          
        # 파일 저장  
        result = write_to_md_file(full_output, output_path)  
          
        if isinstance(result, bool) and result:  
            print(f"청크 {i+1}/{len(chunks)} 저장 완료: {output_path}")  
        else:  
            print(f"오류 발생: {result}")  
      
    print(f"\n총 {len(chunks)}개의 파일로 분할 저장 완료!")  
```
