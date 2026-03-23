# AI Agent 레퍼런스 선정 가이드 — 핵심 요약

> **원본**: `표준 선정 가이드_260323.docx`  
> **부제**: 벤치마크·비용 분석·호환성 검증 기반 기술 선정 원칙  
> **기준일**: 2026.03  
> **문서 성격**: 권고(Recommended) 수준 — 프로젝트별 자율 선택 가능

---

## 1. 문서 개요

LLM 기반 AI Agent 시스템을 구성하는 **기술 영역별 선정 기준, 비교 데이터, 의사결정 프레임워크**를 제공하는 문서이다. 특정 조직에 종속되지 않으며, 클라우드·온프레미스·하이브리드 환경 모두를 다룬다.

**핵심 문제의식**: AI Agent 기술 스택의 조합이 수백 가지에 달하고, 개별 기술이 독립적이지 않아(LLM → Memory → Framework → Vector DB 등 연쇄 영향) 조합 수준의 호환성 검증이 필수적이다.

**적용 범위**: LLM, Framework, Tool 연동(FC/MCP), Agent 간 통신(A2A), Memory, Data Store, RAG Pipeline, Guardrails, Observability

---

## 2. 기술 선정 프레임워크 (선행 조건)

3~11장의 입력 조건이 되는 세 가지 전제로, **개별 영역 선정 전에 반드시 확정**해야 한다.

### 2.1 Agent Topology
| 유형 | 설명 |
|------|------|
| **Single Agent (SAS)** | 단일 LLM이 추론·계획·도구 호출·검증을 하나의 루프에서 처리. 내부 워크플로우 패턴 적용 가능하나 외부 조율 계층 없음 |
| **Multi-Agent (MAS, Centralized 표준)** | 단일 Orchestrator가 하위 전문 Agent를 선택·호출·결과 통합 |
| **비표준 (Collaborative, Decentralized)** | PoC·연구 목적으로 참조 가능. 프로덕션 시 예외 사유 명시 필요 |

### 2.2 배포 환경 유형
| 환경 | 핵심 특성 | 기술 선정 영향 |
|------|-----------|----------------|
| **클라우드** | 관리형 서비스, 자동 스케일링 | 상용 LLM API 즉시 사용, 관리형 Vector DB 활용 가능 |
| **온프레미스** | 데이터 외부 유출 없음, 직접 운영 | 오픈소스 LLM 셀프호스팅(vLLM 등) 필수, GPU 서버 확보 필요 |
| **하이브리드** | 민감 데이터 온프레미스, 비민감 클라우드 | 데이터 분류 기준 사전 정의, LLM 호출 경로 분기 설계 |

### 2.3 기술 영역 간 의존 관계
- LLM 컨텍스트 윈도우 → Memory 전략에 영향
- Framework 선택 → MCP 지원 방식·Memory 백엔드 제한
- Embedding 모델 변경 → Vector DB 인덱스 전면 재구축 필요
- **한 영역의 결정이 다른 영역의 선택지를 제한하거나 변경 비용을 발생시킴**

---

## 3. LLM 선정

전체 시스템의 성능·비용·아키텍처를 결정짓는 **가장 영향력 있는 선택**이다.

### 3.1 9개 선정 기준
| # | 기준 | 핵심 포인트 |
|---|------|-------------|
| 1 | **비용** | Input/Output 토큰 단가 + Prompt Caching/Batch API 등 보정 요소(±30~50% 변동) |
| 2 | **성능·정확도** | 공인 벤치마크 참고 + **자사 업무 데이터 기반 평가 필수**. Tool Use(FC) 정확도 별도 평가 |
| 3 | **응답 지연** | TTFT 2초 이내 권장(실시간 채팅). 경량 모델 가장 빠름, Reasoning 모델 가장 느림 |
| 4 | **한국어 품질** | KMMLU 등 벤치마크 + 자연스러움·도메인 지식·환각 빈도 정성 평가 |
| 5 | **컨텍스트 윈도우** | 128K이면 대부분 충분. 200K 초과 시 장문 프리미엄 요금 vs RAG 구축 비용 트레이드오프 |
| 6 | **데이터 보안** | 배포 환경별 보안 구성, Prompt Injection 저항성, 데이터 학습 활용 여부 확인 |
| 7 | **멀티모달** | 프로젝트 입출력 요구사항 먼저 정의 후 후보 필터링 |
| 8 | **Reasoning 모델** | 고복잡도 전용 티어로 배치, 3.4절 Model Routing에서 선별 적용 |
| 9 | **API 안정성/SLA** | Provider별 Deprecation 주기 차이, Multi-Provider Failover 설계 필수 |

### 3.2 의사결정 트리 (3단계 필터링)
1. **분기 1 — 배포 환경 필터**: 데이터 외부 전송 가능 여부 → 온프레미스/상용 API/하이브리드
2. **분기 2 — 서비스 규모 필터**: 검토할 티어 결정 (온프레미스: GPU 규모별, 상용 API: 트래픽·비용 규모별)
3. **분기 3 — 요구사항 매칭**: 필터 조건 먼저 적용 → 비교 조건으로 최종 선택

### 3.3 모델 비교표
- **상용 모델**: OpenAI, Anthropic, Google, xAI 등의 플래그십/경량/Reasoning 모델 비교
- **오픈소스 모델**: DeepSeek(MIT), Qwen(Apache 2.0), LLaMA(Community License) 등 비교
- **한국어 품질 상세**: 등급(상/중/하)별 KMMLU 점수 및 정성 평가 근거

### 3.4 모델 조합 전략 (Model Routing)
| 패턴 | 설명 | 적합 상황 |
|------|------|-----------|
| **패턴 1: 복잡도 기반 라우팅** | 경량 모델이 1차 분류, 복잡 요청만 플래그십 전달 | 가장 일반적, 비용 대폭 절감 |
| **패턴 1-A: 3단계 라우팅** | 경량 → 범용 → Reasoning | Reasoning 티어 비율 5~15% 이내 통제 |
| **패턴 2: 태스크 유형별 고정** | 태스크별 최적 모델 고정 배정 | 구현 단순, 비용 예측 가능 |
| **패턴 3: 비용 상한 기반 Fallback** | 예산 상한 도달 시 저비용 모델 전환 | 비용 통제 명확 |

### 3.5 LLM 교체 전략
- **원칙**: "최신 모델이므로"가 아니라 "서비스 목표에 더 적합하므로" 교체
- **트리거**: 성능 저하 관측, 비용 구조 변화, Provider Deprecation, 보안·규제 변경 등
- **연쇄 영향 점검**: Framework 호환성, Guardrails, Prompt 재검증, Observability 기준 재설정

---

## 4. Agent Framework 선정

프로젝트 중반 교체 시 사실상 **전면 재작성에 가까운 비용** 발생.

### 비교 대상 (2026.03 기준)
| Framework | 핵심 특성 |
|-----------|-----------|
| **LangGraph** | 그래프 기반 Stateful, Durable Execution, HITL 네이티브. 프로덕션 준비도 최상 |
| **CrewAI** | 역할 기반 직관적, 빠른 PoC. MCP/A2A 지원, MIT 라이선스 |
| **Google ADK** | A2A 네이티브 지원, Gemini 최적화. 2.0 Alpha에서 그래프 기반 도입 |
| **OpenAI Agents SDK** | 경량 코드 우선, OpenAI 네이티브. LiteLLM 확장으로 멀티 Provider(베타) |
| **MS Agent Framework** | Azure 생태계 통합, AutoGen 기반. 프리뷰 단계 |

### 의사결정 분기 (순차 적용)
1. **분기 1**: 독립 Agent 간 협업(A2A) 필요? → Yes: Google ADK 우선
2. **분기 2**: 조건 분기·루프·비동기 HITL·장기 실행 필요? → Yes: LangGraph 우선
3. **분기 3**: 2~4주 내 PoC 최우선? → Yes: CrewAI 또는 OpenAI Agents SDK
4. **분기 4**: LLM Provider/생태계 제약? → OpenAI 전용: Agents SDK, Azure: MS Agent Framework

---

## 5. Tool 연동 방식 (Function Calling / MCP)

### FC vs MCP 핵심 차이
- **FC (Function Calling)**: 기본(default). Agent가 도구 스키마를 직접 보유, LLM이 호출 결정
- **MCP (Model Context Protocol)**: 도구를 표준화하여 노출하는 서버 계층. FC와 **보완 관계**
- MCP는 도구 수 5개 이상, 다수 Agent 간 도구 공유, 중앙 보안 통제 필요 시 검토

### MCP 도입 손익분기점
- 도구 20개 이상 + 3개 이상 Agent 공유 + 도구 변경 월 2회 이상 → MCP TCO 우위 가능성
- 동적 도구 검색(Dynamic Tool Retrieval) 적용 시 토큰 비용 절감 효과 극대화

### 레거시 시스템 연동
- 기존 REST API 래퍼가 있으면 MCP 래핑 필요성을 재사용 가능성 기준으로 판단
- API 유형별(REST/SOAP/RPC/DB) 래핑 패턴 및 공수 예시 제공

---

## 6. Agent 간 통신 (A2A)

### 도입 판단 핵심
- **"Agent들이 독립적으로 배포·운영되는가?"가 유일한 판단 기준**
- Framework 내부 sub-agent 제어 구조는 A2A 불필요
- Q1~Q3 모두 "예"인 경우에만 도입 검토 → **대부분의 프로젝트에서는 불필요**

### 현실적 권고
- 생태계 초기 단계(2025.04 발표) → **PoC 우선 접근** 권고
- 대안: REST 직접 호출, API Gateway, Event-Driven(Kafka/Redis Streams)
- 전환 트리거: 협업 Agent 5개 이상, Agent Card 자동 발견 필요, 외부/파트너사 협업

---

## 7. Memory 전략

### 3단계 의사결정 플로우
| Phase | 대상 | 핵심 판단 |
|-------|------|-----------|
| **Phase 1** | 단기(Short-Term) 컨텍스트 관리 | 전체 히스토리/윈도우/요약/RAG 기반 선택 |
| **Phase 2** | 장기(Long-Term) 메모리 유형 | Semantic/Episodic/Entity/User Preferences 복수 선택 |
| **Phase 3** | 관리 도구 | Mem0, Zep, LangMem, Framework 내장, 자체 구현 |

### 장기 메모리 유형
| 유형 | 저장 대상 | 검색 전략 |
|------|-----------|-----------|
| **Semantic** | 사실·지식 임베딩 | 벡터 유사도 검색 |
| **Episodic** | 과거 작업 경험 (맥락+결과+교훈) | 하이브리드(벡터+키워드) |
| **Entity** | 엔티티 속성·관계 | 구조화 쿼리 (RDBMS/Graph DB) |
| **User Preferences** | 사용자 설정·선호 | 구조화 쿼리 |

### 핵심 설계 포인트
- **승격 기준**: 모든 단기 메모리를 장기로 저장하면 노이즈 축적 → 선별 승격 필수
- **감쇠 전략**: TTL만으로는 불충분, 세분화된 감쇠 메커니즘 설계
- **충돌 해소**: 장기 메모리 정보와 새로운 입력이 모순될 때의 해소 전략 필수(MUST)

---

## 8. Data Store 선정

### 8.1 Vector DB
- **시작점**: pgvector(PostgreSQL 이미 사용 중), Chroma(프로토타이핑 전용)
- **프로덕션 전용**: Milvus(대규모), Qdrant(성능·필터링), Pinecone(관리형), Weaviate(하이브리드 검색)
- **pgvector 전환 트리거**: 5M 벡터 초과, 필터+벡터 검색 레이턴시 SLA 초과, GPU 가속 필요

### 8.2 Embedding 모델
- **핵심 원칙**: "최고의 Vector DB도 나쁜 임베딩을 보상하지 못한다"
- 모델 변경 시 **기존 벡터 전체 재생성 필수** → 초기 신중 선정
- 프로덕션 권장: 384~1024 차원으로 시작, 부족 시 차원 상향
- 도메인 특화 Fine-tuning으로 Recall@10 기준 10~30% 향상 가능

### 8.3 Graph DB
- **대부분의 Entity Memory는 RDBMS 외래 키+조인으로 충분**
- Graph DB 필요 조건: 3홉 이상 관계 탐색 + 실시간 관계 갱신 동시 요구

---

## 9. RAG Pipeline

### 구성요소별 권장
| 구성요소 | 기본 권장 | 비고 |
|----------|-----------|------|
| **Chunking** | Recursive Character Splitting (512토큰, 오버랩 50~100) | 전문 도메인은 Adaptive Chunking 검토 |
| **검색 방식** | 하이브리드(Dense+Sparse) + 리랭킹(Cross-Encoder) | 프로덕션 베스트 프랙티스 |
| **검색 도구** | LangChain(LangGraph 선정 시 자연스러운 통합), LlamaIndex(데이터 중심 RAG 특화) | - |

### 프로덕션 검색 파이프라인 3단계
1. **하이브리드 검색**: Dense(벡터) + Sparse(BM25) → Recall 극대화
2. **리랭킹**: Cross-Encoder로 후보 재정렬 → Precision 극대화
3. **Context Assembly**: 상위 K개만 LLM 컨텍스트에 조립 → 비용·품질 균형

---

## 10. Guardrails

### 다층 방어 원칙 (Defense-in-Depth)
1. **1계층**: System Prompt 수준 지시
2. **2계층**: 코드 레벨 검증(입력 검증, 출력 파싱)
3. **3계층**: 전용 Guardrails 도구 ← **이 장의 선정 대상**

### 도구 선정 핵심
| 도구 | 강점 | 적합 시나리오 |
|------|------|---------------|
| **Guardrails AI** | Pydantic 기반 출력 검증, Hub 100+ Validator | 구조화 출력 보장, PII 탐지 |
| **NeMo Guardrails** | Colang DSL로 대화 흐름 제어, 파이프라인 오케스트레이션 | 멀티턴 대화 제어, Prompt Injection 방어 |
| **LlamaGuard** | 문맥 기반 유해 콘텐츠 분류 전용 LLM | 온프레미스 유해 콘텐츠 필터링 |

- **한국어 환경**: 대부분 영어 중심 최적화 → 한국어 성능 별도 검증 필수
- **Prompt Injection 방어**: 예외적으로 이중 적용(NeMo + LlamaGuard) 권장

---

## 11. Observability

### 프로젝트 단계별 수준
| 단계 | 수준 | 도구 |
|------|------|------|
| **단계 1** (로컬 개발) | logging/print | 별도 도구 불필요 |
| **단계 2** (팀 개발) | Framework 내장 트레이싱 | LangSmith, Langfuse 등 |
| **단계 3** (프로덕션) | 전용 도구 + 비용·품질 모니터링 | 트레이싱 + 보조 레이어 조합 |
| **단계 4** (온프레미스) | 셀프호스팅 | Langfuse(MIT), Arize Phoenix(EL2.0) |

### 주요 도구 비교
| 도구 | 포지셔닝 | 라이선스 |
|------|----------|----------|
| **LangSmith** | LangChain 네이티브, 가장 깊은 트레이싱 | 상용 SaaS |
| **Langfuse** | 오픈소스 셀프호스팅, OpenTelemetry 호환 | MIT |
| **Helicone** | AI Gateway 프록시 방식, 비용 추적 강점 | Apache 2.0 |
| **Arize Phoenix** | 임베딩 드리프트·RAG 품질 분석 특화 | Elastic License 2.0 |
| **OTel + SigNoz** | 벤더 중립, 기존 인프라 통합 | Apache 2.0 / MIT |

---

## 12. Orchestration 전략 — 아키텍처 유형별 기술 조합

| 유형 | Framework | Tool 연동 | Memory | Guardrails | Observability |
|------|-----------|-----------|--------|------------|---------------|
| **단일 Agent** | LLM API 직접 / LangGraph | FC 위주 | 리스트/내장 메모리 | 코드 레벨 | logging+verbose |
| **Sequential** | CrewAI / LangGraph | Agent별 Tool, MCP 권장 | 공유 State / 메시지 패싱 | Agent별+전체 출력 검증 | Agent별 트레이싱 |
| **Graph/State** | LangGraph | MCP + 동적 Tool 선택 | Checkpointer | 분기별 검증 + HITL | 노드별 트레이싱 |
| **Hierarchical** | LangGraph / AutoGen | MCP + A2A | 전역+로컬 상태 분리 | Worker 권한 제한 | **분산 트레이싱 필수** |
| **Collaborative** | AutoGen GroupChat | Agent별 도구 + A2A | 대화 컨텍스트 유지 | 라운드 제한, 유해 필터링 | 대화 흐름 전체 트레이싱 |

---

## 13. 유스케이스별 기술 스택 권장안

| 유스케이스 | 주요 구성 |
|------------|-----------|
| **사내 문서 QA Agent** | 단일 Agent + RAG Pipeline + Vector DB |
| **대용량 실시간 데이터 처리** | 그래프 기반 멀티 Agent + 스트리밍 |
| **고객 응대 Agent** | Sequential/Graph + Memory(개인화) + Guardrails |
| **코드 생성/리뷰 Agent** | 멀티 Agent + Reasoning 모델 + 코드 보안 Guardrails |

---

## 핵심 의사결정 원칙 요약

1. **Topology → 배포 환경 → 의존 관계 순서로 전제 조건을 먼저 확정**한 후 개별 기술 선정에 진입
2. **벤더 벤치마크보다 자사 데이터 PoC 결과**를 기반으로 최종 판단
3. **단일 모델/도구로 시작하고, 운영 데이터 기반으로 점진 확장** (오버엔지니어링 방지)
4. **조합 수준의 호환성 검증** — 개별 기술이 아니라 전체 스택 관점에서 검토
5. **교체 비용을 고려한 선택** — "이 선택이 틀렸을 때 교체 비용을 감당할 수 있는가?" 자문
6. **모든 비교표·가격표는 기준일(2026.03)이 명시**되어 있으며, 도입 시점에 최신 정보 재확인 필수
