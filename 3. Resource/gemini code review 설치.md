

https://blog.naver.com/pjt3591oo/224061833898

https://github.com/pjt3591oo/gemini-code-assist-test

  

https://goddaehee.tistory.com/373

  

  

✦ 핵심 변경 사항:

   1. 공식 구조 준수: 사용자가 제공해주신 have_fun, memory_config, code_review 구조로 전면 수정했습니다.

   2. comment_severity_threshold: HIGH: 아까 논의한 대로 가장 꼼꼼하게 설계 원칙을 보도록 설정했습니다.

   3. max_review_comments: -1: 학습 목적이므로 Gemini가 모든 잠재적 개선 사항에 대해 자유롭게 의견을 낼 수 있도록 제한을

      없앴습니다.

   4. system_instructions: 앱이 styleguide.md를 읽어오는 것은 기본이지만, '한국어 리뷰'와 '오브젝트 설계 전문가'

      페르소나를 더 강력하게 주입하기 위해 system_instructions를 유지했습니다.

   5. ignore_patterns: ignore_patterns는 사용자 템플릿의 형식을 그대로 따르면서 분석에서 제외할 불필요한 빌드 파일들을

      담았습니다.