# 쥬라기원시전2: 더랭커 버전변경기

쥬라기원시전2: 더랭커의 데이터 아카이브인 `jw2_01.trc` 안에는 게임 버전 날짜가 두 곳에 박혀 있다. 연도 2바이트가 한 곳, 월과 일 1바이트씩이 다른 곳이다. 그 자리의 오프셋을 미리 찾아 두고 원하는 날짜로 바꿔 쓰는 도구다. 입력 폼을 갖춘 GUI로 만들었다.

**호환 버전**: 쥬라기원시전2: 더랭커 v2003.12.29

<p>
  <img src="docs/screenshots/screenshot-1.png" width="306" alt="날짜 입력 화면">
</p>


## 사용 방법

Releases에서 받아 압축을 풀고 실행하면 파일 선택창이 먼저 뜬다. 여기서 게임 폴더의 `jw2_01.trc` 를 고르면 현재 버전 날짜가 읽혀서 세 칸에 채워진다.

칸을 클릭하면 입력 상태로 바뀌고 숫자를 치면 된다. 다른 곳을 클릭하면 입력이 끝난다. 연도는 9999, 월은 12, 일은 31을 넘길 수 없다.

OK를 누르면 trc 파일이 그 자리에서 고쳐진다. 되돌릴 방법이 없으니 실행 전에 `jw2_01.trc` 를 백업해두는 게 좋다.

오프셋을 고정값으로 박아뒀기 때문에 원본 버전이 2003.12.29 인 trc에서만 동작한다.


## 구현 원리

**연도는 두 바이트라서 잘라 써야 했다.** 연도만 0에서 9999까지 들어가므로 한 바이트에 담기지 않는다. 16진수로 바꾼 뒤 뒤에서부터 두 글자씩 떼어내 각각을 다시 10진수로 되돌리면 그게 곧 바이트 값이 된다. 2011이면 `07DB` 가 나오고 이걸 `DB`, `07` 로 잘라 낮은 자리부터 쓴다. 게임메이커 8에는 진법 변환 함수가 없어서 16진수 변환은 직접 짠 스크립트를 쓴다.

```gml
// sk_cut_hex.gml - 뒤에서 두 글자씩 떼어 바이트로 만든다
stringeses = string_copy(num1, length-1, length)
num1 = string_delete(num1, length-1, length)
stringsrfg[j] = sk_dec_conversion(stringeses)
```

월과 일은 각각 12와 31을 넘지 않으니 변환 없이 한 바이트로 그대로 쓴다.

**쓰기는 파일 전체를 복사하면서 해당 바이트만 갈아끼우는 식이다.** trc를 처음부터 끝까지 한 바이트씩 읽어 `tmp.trc` 에 옮기다가, 버전 날짜 자리에 오면 원본 대신 입력값을 써 넣는다. 다 옮기고 나면 원본 자리에 덮어쓰고 임시 파일을 지운다.

```gml
for(i=0; i!=file_size; i+=1)
{
  file_bin_seek(open_file, i)
  file_message = file_bin_read_byte(open_file)

  file_bin_seek(save_file, i)
  file_bin_write_byte(save_file, file_message)

  if i=1117006
  {
    sk_cut_hex(sk_hex_conversion(real(obj_year.vares)))
    file_bin_seek(save_file, i+1)
    file_bin_write_byte(save_file, real(stringsrfg[0]))
    file_bin_seek(save_file, i+2)
    file_bin_write_byte(save_file, real(stringsrfg[1]))
    i+=2
  }
  ...
}
```

소스에 적힌 `1117006` 은 10진수로, `0x110B4E` 와 같은 값이다.

읽을 때도 같은 모양이다. 포맷을 해석하지 않고 파일 전체를 훑다가 미리 알아둔 인덱스에 닿으면 그 바이트를 집는다. 오프셋이 그대로 박혀 있는 만큼 다른 버전의 trc에서는 엉뚱한 자리를 건드리게 된다.


## 파일

| 경로 | 내용 |
|---|---|
| `source/jw2-version-changer.gmk` | 원본 프로젝트 파일 |
| `source/split/` | GmkSplitter로 분해한 텍스트 트리 |
| `docs/screenshots/` | 스크린샷 |
| Releases | 실행 파일과 사용 설명 |


## 크레딧

한글 출력 스크립트(`source/split/Scripts/한글드로우/`)는 게임메이커 커뮤니티의 김게맛(sodium031)님이 만든 것이다.


## 라이선스

zlib 라이선스다. 자세한 내용은 [LICENSE](LICENSE) 에 있다. 함께 들어 있는 것 중 다른 사람이 만든 라이브러리는 각자의 라이선스를 따른다.
