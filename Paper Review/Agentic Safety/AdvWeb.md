요약

- VLM based web agent 기술이 발전하고 있으나, 악의적인 공격에 대한 안전성, 보안 문제가 충분히 탐구 되지 않은 상태이다. 배포하기에 심각한 우려가 제기된다.
    
    → web agent의 취약점을 밝히고 악용해봐야 한다.
    
    → web agent를 대상으로 하는 새로운 black-box 공격 프레임워크 advweb을 제안한다.
    

Keywords

- gpt-4v based vlm web agent
- black-box
- adversarial prompter model training
- Direct Policy Optimization (DPO)

Contribution: adversarial string injection maintains **stealth and control**

- 기존 접근법↔ 본 논문 adversarial string 삽입 방식: 스텔스성과 통제성
    - 공격 전후의 웹사이트의 외관이 변하지 않아 사용자가 변조를 탐지하기 거의 불가능하고,
    - 생성된 adversarial string 내 특정 부분 문자열을 변경하여 공격 목표를 쉽게 수정할 수 있다(예: 다른 회사의 주식을 구매하도록 설정). 이로써 공격의 유연성과 효율성이 크게 향상된다.

문제상황

- LLMs와 VLMs의 발전 → generalist web agents(현실 세계의 웹사이트와 상호작용하며 작업 수행)
    - web agents는 금융, 의료, 전자상거래와 같은 고위험 분야를 포함한 다양한 분야에서 인간의 생산성을 크게 향상할 잠재력을 지님
    - 그러나 adversarial attacks에 대한 robustness 측면에서 보안 과제를 제시한다.
- 최근 연구에서는, 현실 세계로 배포되기 전에 agent의 취약성을 드러내고자, adversarial attacks를 도입함.
    - 기존 접근 방식은 수동으로 공격 프롬프트를 설계하는 데 인적 노력이 필요한 경우와 특정 공격 시나리오에 집중하는 경우에 비용이 많이 들고, 더 효율적이고 적응성 높은 공격 프레임워크 개발에는 한계가 있다.
    - LLM과 VLM에 대한 adversarial attack을 통해 공격 프롬프트를 자동으로 최적화하려는 시도가 있긴 하지만, 이러한 방법은 VLM 기반 에이전트를 대상으로 하는 유연성이 부족하며, 블랙박스 공격 설정에서는 효과적인 전이성을 달성하는 데 어려움을 겪는다.

관련 연구의 한계점

- Adversarial Attack on LLM
    
    - token 기반 공격(token의 이산적 특성으로 인해 공격을 최적화하는 것이 이미지 기반 공격보다 어려움)
        
        - **초기 연구**(Ebrahimi et al., 2018; Wallace et al., 2019; Shin et al., 2020)
            
            특정 반응을 이끌어내거나 유해한 출력을 생성하기 위해, 토큰 수준에서 최적화된 공격 문자열을 사용함. 주로 greedy search이나 gradient information을 활용하여 중요한 토큰을 수정하는 방식
            
        - **ARCA(**Jones et al., 2023)
            
            토큰 수준의 최적화를 정교화하여 여러 토큰을 동시에 교환함으로써 효율성을 높임
            
        - **GCG Attack**(Zou et al., 2023)
            
            공격의 효과를 향상하기 위해 긍정적인 responses을 유도하는 방식으로 공격 문자열을 최적화했음
            
        
        → (한계점) token 기반 방법은 종종 가독성이 떨어지며, perplexity-based detectors에 의해 쉽게 탐지된다.
        
    - 다른 방법
        
        - **AutoDan**(Liu et al., 2024c) hierarchical genetic algorithm을 사용하여 적대적 프롬프트의 스텔스성을 높임
        - **AmpleGCG**(Liao & Sun, 2024)와 **AdvPrompter**(Paulus et al., 2024) generative models을 직접 사용해 gradient based optimization에 의존하지 않고 adversarial suffixes를 생성
        
        →(한계점) 유해 프롬프트에 대해 긍정적 반응을 이끌어낸다는 간단한 목표에 주로 초점, 특히 vlm based web agent에서 더 복잡한 공격 목표를 달성하기 어렵다.
        
    
    → (개선안) 1. 복잡한 목표 다루기(ex. 구매 결정 조작) 2. 스텔스성, 통제성 유지
    
- Web agent
    
    - 기존
        
        - Nakano et al., 2021; Wu et al., 2024b
            
            retrieval capabilities가 추가된 LLM을 사용해 generalist web agent를 구축한다.
            
        - 최근 접근 방식(Yao et al., 2022; Zhou et al., 2023; Deng et al., 2024
            
            웹페이지의 **HTML 입력**을 직접 처리하여 인간의 지시에 기반해 시뮬레이션 혹은 현실적인 웹 환경에서 작업을 수행할 수 있는 웹 에이전트를 개발한다.
            
        
        → (한계점) HTML 콘텐츠는 사람의 웹 탐색에 사용되는 시각적 정보에 비해 **정보 밀도**가 낮고 noise가 포함되어 있어 작업 성공률이 낮고 실용적인 배포가 어려운 문제가 있다.
        
    - 개선
        
        - SeeAct(Zheng et al., 2024) generalist web agent 프레임워크
            
            웹페이지의 스크린샷을 입력으로 사용하는 두 단계의 파이프라인을 도입 → 추론 능력 및 작업 완수 성능 향상
            
    
    → Advweb은 SeeAct를 주요 공격 대상을 삼음.
    
    → 그러나, 웹 페이지 스크린샷 or html 콘텐츠를 입력으로 하는 다른 웹 에이전트에도 쉽게 적용가능
    
- Existing Attacks against Web Agents
    
    - Yang et al. (2024)과 Wang et al. (2024)
        
        백본 모델을 backdoor attacks으로 미세 조정하여 웹 에이전트가 잘못된 결정을 내리도록 유도하는 방법을 탐구했다.
        
    - Wu et al., 2024b; Liao et al., 2024
        
        웹 콘텐츠에 악의적인 지시를 삽입하여 웹 에이전트가 간접적인 프롬프트를 따르도록 유도해 부정확한 출력이나 민감한 정보를 유출하도록 하는 방법을 사용했다.
        
        → 악의적 지시는 수동으로 설계되고 heuristics에 의존하여 작성되기 때문에 확장성과 유연성에 한계가 있다.
        
    - Wu et al.(2024a) 웹 에이전트를 속이기 위한 auto 적대적 입력 최적화 방법을 도입
        
        **→ 기울기 기반 최적화**를 위한 화이트박스 접근이 필요하거나, 여러 **CLIP 모델**에 공격을 전이하는 데 한계를 보였다.
        
    
    → AdvWeb은 black box **설정**에서 웹 에이전트를 공격하는 것을 목표로 한다.
    
    RL 을 활용해 성공/실패 공격 시도의 피드백을 통합하여 학습→ 웹에이전트를 공격하는 생성 모델을 훈련한다.
    

제안

- generalist web agents 취약성 탐구 목적의 black box 제어 공격 프레임워크 AdvWeb
- 방식
    - 웹 페이지에 보이지 않는 adversarial strings를 삽입하여, 에이전트가 특정 유해한 적대적 행동을 수행하도록 유도한다.
        - (ex) 잘못된 금융 거래나 부적절한 주식 구매
    - 피해 에이전트의 블랙박스 피드백을 기반으로 RL을 활용하여 adversarial strings를 최적화하는 **두 단계의 훈련 방식**을 제안한다.
        - **Direct Policy Optimization (DPO)** 기법을 적용하여 성공 및 실패한 공격 시도의 피드백을 학습에 활용함으로써, 공격의 유연성과 효율성을 높였다.
        - 특히 AdvWeb은 특정 사용자 요청에 대해 다른 회사나 작업을 대상으로 하도록 적대적 문자열을 쉽게 조정할 수 있어 재최적화가 필요하지 않다.
![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c81e1106-6576-44de-83bf-6e68d6e7fb70/e7861476-89e0-48c8-9932-0fe94588caf2/image.png)


방법론

Targeted Black-box Attack against Web Agents

- Preliminaries on Web Agent Formulation
    
    <aside> 💡
    
    web agent (예: SeeAct) 의 설계 방식
    
    - Input
        
        - 특정 웹사이트(예: 주식 거래 플랫폼)
        - 작업 요청 **T**(예: "Microsoft 주식 한 주를 구매하라")
    - action
        
        - 웹사이트에서 작업 **T**를 성공적으로 완료하기 위해 실행 가능한 **행동의 연속 {a1, a2, ..., an}**을 생성
        - action 결정법
            - 각 시간 단계 t에서) 현재 환경의 관측치 **st**
            - agent는 st 이전에 실행된 행동 **At = {a1, a2, ..., at-1}, 작업 T**를 기반으로 행동 **at**을 결정한다. </aside>
    - seeact의 경우, 관측치 st는 2개의 구성요소
        
        (1) 웹페이지의 **HTML 콘텐츠 ht**
        
        (2) (1)에 대응하는 **렌더링된 스크린샷 it = I(ht)**
        
    - VLM policy model(예: GPT-4V): Π
        
    - agent가 policy model을 통해 만드는 action: (관측치, 작업, executed actions)
        
        ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c81e1106-6576-44de-83bf-6e68d6e7fb70/411ccc3b-eb68-41aa-aac7-1d6357dd9649/image.png)
        
        agent는 st 이전에 실행된 행동 **At = {a1, a2, ..., at-1}, 작업 T**를 기반으로 각 행동 **at**을 결정한다.

<aside> 💡

each action

- **각 행동 at은 (ot, rt, et)**이라는 triplet으로 구성됨
    
    - ot는 수행할 작업
    - rt는 작업에 대응하는 argument
    - et는 목표 HTML 요소
    
    예를 들어, **거래 웹사이트**에서 **주식 이름을 입력**하기 위해,
    
    에이전트는 **주식 이름인 Microsoft(rt)**를 주식 **입력 텍스트 상자(et)**에 입력하는 동작(ot)을 수행할 것이다.
    
- 행동 at이 수행되면 웹사이트는 이에 따라 업데이트됨
    
- 에이전트는 작업이 완료될 때까지 이 과정을 반복함 </aside>
    

(간결성을 위해 특별히 명시되지 않는 한 이후 식에서는 시각 t 생략)

- Threat Model
    
    - Attack Objective
        
        - 에이전트의 행동을 targeted 공격으로 고려함
            - agent action(at = (ot,rt,et)) → targeted adversarial action: **a_adv = (o, r_adv, e)**으로 변경
                
            - 이때 에이전트는 동일한 작업을 수행하지만 잘못된 **악의적 인수**를 사용하게 된다.
                
                예를 들어, **Microsoft(r) 주식**을 구매하라는 요청이 **NVIDIA(r_adv) 주식**을 구매하도록 공격받으면, 사용자는 큰 재정적 손실을 입을 수 있다.
                
    - Environment Access, Attack Scenarios
        
        - black box 환경
            
            - 이유: 대부분 웹 에이전트가 vlm 기반이다. → 에이전트 프레임워크, 백엔드 모델 가중치, 또는 백엔드 모델의 로그에 접근할 수 없다.
        - Attack Scenarios
            
            <aside> 💡
            
            - 공격자 → 웹사이트 html 콘텐츠 h에 접근
                
                ```
                                  + html 콘텐츠 변경 가능(h → h_adv)                   
                ```
                
            
            </aside>
            
        - 위 설정은 현실적인 공격 시나리오임
            
            예를 들어, 악의적인 웹사이트 개발자가 정기적인 유지 보수나 웹사이트 업데이트 과정에서 HTML 콘텐츠를 의도적으로 수정함으로써 이익을 얻을 수 있다.
            
            또한, 악성 라이브러리를 사용한 웹페이지 개발로 인해 의도치 않게 공격이 발생할 수 있으며, 최근 CISA 보고서(Synopsys, 2024)에 따르면 이러한 라이브러리가 숨겨진 취약점을 초래할 수 있다.
            
        - Attack Constraints
            
            - 공격은 **stealth** 와 **control** 을 모두 만족해야 한다.
                
            - (1) stealth 충족
                
                - **렌더링된 스크린샷 i = I(h)**는 HTML 콘텐츠 h에 의해 영향을 받으므로 사용자가 공격을 탐지하지 못하게 렌더링된 이미지는 변경되지 않아야 한다.
                    
                - 즉, HTML 콘텐츠의 수정이 사용자에게 보이는 시각적 정보에 영향을 주지 않도록 한다.
                    
                    <aside> 💡
                    
                    - **I(h) = I(h_adv)** </aside>
            - (2) control 충족
                
                - 공격자는 적대적 프롬프트**만** 수정하여 **새로운** 목표에 맞는 공격을 **쉽게** 수행할 수 있어야 하며, 에이전트와의 추가적인 상호작용이나 최적화는 필요하지 않아야 한다.
                
                <aside> 💡
                
                목표 행동a_adv = (o, r_adv, e) → a'_adv = (o, r'_adv, e) 변경시,
                
                공격자는 **D(h_adv, r_adv, r'_adv)**라는 결정론적 함수를 사용 가능
                
                - 기존의 적대적 HTML 콘텐츠 h_adv
                - 원래의 목표 인수 r_adv
                - 새로운 목표 인수 r'_adv
                - 새로운 적대적 HTML 콘텐츠 h'_adv
                
                예를 들어, Microsoft(r) 주식을 구매해야 하는 요청이 NVIDIA(r_adv) 주식을 구매하도록 성공적으로 공격한 h_adv의 경우, h'_adv = D(h_adv, "NVIDIA", "Apple")을 통해 목표를 Apple(r'_adv)로 변경하여 유연하게 공격을 제어할 수 있다.
                
                </aside>
                
        - Challenges of Attacks against Web Agents (총정리)
            
            1. **Discrete space with stealthiness and controllability constraints**
                
                적대적 HTML 콘텐츠 h_adv의 결정 공간은 이산적이며 최적화가 본질적으로 어렵다. 또한, 공격은 스텔스성과 통제성이라는 두 가지 중요한 제약 조건을 충족해야 하며, 이는 최적화 경관을 복잡하게 만든다.
                
            2. **Black-box attack**
                
                블랙박스 환경에서는 모델의 매개변수나 기울기에 접근할 수 없으며, 사용할 수 있는 피드백은 에이전트가 적대적 입력에 반응한 결과뿐이다. 이 제약 조건은 기존의 최적화 기법을 비효과적으로 만들며, 기울기 정보를 사용하지 않고도 공격 공간을 효율적으로 탐색할 수 있는 보다 정교한 방법이 필요하다.
                
            3. **Limited efficiency and scalability**
                
                기존 접근법은 ****seed prompt를 설계하거나 공격 시나리오를 설계하는 데 많은 수작업이 필요하다. 이러한 수작업 과정은 특히 다양한 복잡한 작업을 대상으로 할 때 확장성과 유연성을 제한한다. 이러한 도전 과제를 극복하기 위해, **RL** 기반의 새로운 공격 프레임워크를 제안한다. 이 프레임워크는 이산 최적화 문제를 해결하는 동시에 스텔스성과 통제성을 보장하고, 블랙박스 공격 시나리오를 효과적으로 다룰 수 있도록 설계되었으며, 수작업의존성을 줄여 효율성을 높인다.
                
- AdvWeb: Controllable Black-box Attacks on Web Agents
    
    1. 스텔스성과 통제성 제약을 준수하도록 설계된 **advarsarial HTML 콘텐츠 h_adv의 검색 공간을 줄인다.**
        
        → 생성된 적대적 콘텐츠가 사용자에게 탐지되지 않도록 하고, 공격 목표를 다시 최적화할 필요 없이 유연하게 수정할 수 있도록 한다.
        
    2. **웹 에이전트의 응답으로부터 얻은 긍정적 및 부정적 피드백을 활용한다. RLAIF을 사용하여 적대적 문자열 생성 과정을 효율적으로 최적화함**
        
        → 웹 에이전트를 대상으로 한 타겟형 블랙박스 공격에서 학습을 강화하고, 복잡한 블랙박스 웹 에이전트의 공격 패턴에 적응하도록 한다.
        
    3. **자동**으로 적대적 문자열을 생성하는 **생성 모델**을 사용
        
        → 수작업 의존성 최소화 → 공격 유연성, 확장성, 효율성 향상
        
- AdvWeb 주요 구성 요소
    
    (1) 자동 공격 및 피드백 수집
    
    (2) AdvWeb에서의 적대적 프롬프터 모델 학습
    
- (1) Automatic Attack and Feedback Collection
    
    - **Adversarial HTML Content Design**
        
        <aside> 💡
        
        benign HTML 콘텐츠 h
        
        - 프롬프트 q의 일부를 삽입
        
        = 적대적 버전 h_adv 제작
        
        </aside>
        
        적대적 HTML 콘텐츠 h_adv의 검색 공간:
        
        고차원&이산적으로 최적화 어려움, 스텔스성과 통제성 제약조건
        
        → h_adv의 검색 공간을 **특정 방식**으로 줄인다.
        
        <aside> 💡
        
        - 삽입된 프롬프트 q는 웹사이트에서 렌더링되지 않도록 특정 HTML 필드나 속성(예: “aria-label” = q) 내에 숨겨진다.
            
        - 또한, q를 다양한 목표 행동으로 전이 가능하게 만들기 위해 **target argument**에 대한 **placeholder**(예: “{target argument}”)를 삽입하고,
            
            모델이 먼저 placeholder가 포함된 프롬프트 템플릿을 생성한 후 목표에 맞는 인수를 채우도록 훈련한다.
            
        - 검색 공간을 줄이기 위해 삽입 위치를 **ground truth HTML 요소**인 **e**에 고정한다.
            
        
        이러한 HTML 숨김 기법과 placeholder를 활용하여 검색 공간을 효과적으로 줄이고, 스텔스성 제약을 만족하며, 통제성을 위한 최적화 프로세스를 단순화한다.
        
        </aside>
        
    - **Automatic Attack and Feedback Collection Pipeline**
        
        - 초기 **RL** 훈련을 위해 긍정적, 부정적 레이블을 가진 다양한 학습 사례가 필요함
            
        - 훈련 데이터의 다양성을 확보하기 위해,
            
            (1) **LLM**을 공격 프롬프터로 활용하여 **다양한 공격 프롬프트 세트 {q_i}**를 생성한다.
            
            (algorithm2 방식)
            
            (2) 생성된 적대적 프롬프트를 사용해 블랙박스 웹 에이전트를 대상으로 공격을 시도한다.
            
            (3) 피드백을 바탕으로 positive signals와 negative signals로 구분한다.
            
            → 이러한 분류된 피드백을 RL 훈련에 활용한다.
            
- (2) Training Adversarial Prompter Model in AdvWeb
    
    ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c81e1106-6576-44de-83bf-6e68d6e7fb70/c7abc6b7-d223-40d5-bdd8-7fd55c349d9b/image.png)
    
    - 다양한 웹 환경을 다루고 공격의 효율성과 확장성을 보장하기 위해,
        
        프롬프터 모델을 훈련하여 적대적 jailbreaking prompt **q**를 생성하고
        
        이를 HTML 콘텐츠에 삽입하도록 한다.
        
    - 다양한 적대적 프롬프트 간의 미묘한 차이를 더 잘 포착하기 위해,
        
        **RLAIF** 학습 방식 적용
        
    - **RLAIF** 은 보통 훈련과정에서 불안정하기 때문에,
        
        강화 학습 이전에 **supervised fine-tuning (SFT)** 단계를 추가하여 훈련을 안정화한다.
        
        AdvWeb의 전체 훈련 프로세스
        
        (1) 긍정적 적대적 프롬프트를 사용한 SFT
        
        (2) 긍정적 및 부정적 프롬프트를 모두 사용하는 강화 학습.
        
        ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c81e1106-6576-44de-83bf-6e68d6e7fb70/ab52761f-eac5-4ff4-8de7-dee7e95d35ab/image.png)
        
        ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c81e1106-6576-44de-83bf-6e68d6e7fb70/c229f688-ea47-421e-b8dc-d82038fd877a/image.png)
        
        ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c81e1106-6576-44de-83bf-6e68d6e7fb70/664f44bf-239b-4912-abf0-67d5b4404662/image.png)
        
- **(1) Supervised Fine-tuning in AdvWeb**
    

SFT 단계에서는 프롬프터 모델 가중치 θ를 최적화하여 **긍정적인 적대적 프롬프트의 가능성을 극대화**한다.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c81e1106-6576-44de-83bf-6e68d6e7fb70/79ca1fd3-659f-4ad2-a212-e2594c2c9475/image.png)

→ 모델이 긍정적 적대적 프롬프트의 분포를 학습

→ RL 단계 안전성 향상 + 모델은 web agent가 목표행동을 성공하도록 유도하는 프롬프트를 잘 생성

- **(2) Reinforcement Learning Using DPO**
    
    - rl 목표: 공격 패턴의 미묘한 차이 포착& 프롬프터를 공격 목적에 더 잘 맞추기 위함.
    - DPO (Direct Preference Optimization)를 사용하여 강화 학습 과정을 안정화함
    - 프롬프터 모델의 가중치 θ 최적화
    
    σ 는 logistic**,** β는 기본 리퍼런스 폴리시인 πref로부터 편차를 조절하는 매개변수
    
    πref는 이전 SFT 모델인 πSFT로 초기화되고 고정된 것
    
    ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c81e1106-6576-44de-83bf-6e68d6e7fb70/ca746127-e2b4-4808-87fa-08943299d393/image.png)
    
    이 최적화 프레임워크는 프롬프터 모델이 반복적으로 파라미터를 세밀하게 조정하여, 타겟 적대적 행동a_adv을 수행하도록 웹 에이전트를 오도할 수 있는 성공적인**adversarial jailbreaking prompts**를 생성할 가능성을 극대화할 수 있도록 한다.
    

실험

- **Victim Web Agent**
    
    - 웹 에이전트 **SeeAct**(Zheng et al., 2024)
        
        - 다양한 독점 VLM(예: GPT-4V, Gemini 1.5)이 구동된다.
        
        <aside> 💡
        
        동작 과정
        
        (1) 작업과 웹페이지 스크린샷을 기반으로 행동 설명("Microsoft"라는 주식 이름을 입력해야 한다”)생성
        
        (2) 이를 해당 HTML 콘텐츠(ht)와 매핑하여 웹 환경과 상호작용("주식 이름 입력" 텍스트 상자에 "Microsoft"를 입력해야 한다)
        
        </aside>
        
- **Dataset and Metrics**
    
    - **Mind2Web** 데이터셋(Deng et al., 2024)
        - LLM/VLM 기반 에이전트를 평가하기 위한 현실 세계의 웹사이트 데이터를 포함한다.
        - 31개의 도메인에서 137개의 웹사이트와 2,350개의 작업이 포함되어 있다.
        - advweb에서는 **잠재적으로 심각한 결과를 초래할 수 있는 중요한 이벤트가 포함된 작업에 초점을 맞추어**, 4개의 도메인에서 440개의 작업을 선택하고 이를 학습용 240개와 테스트용 200개 작업으로 나눈다.
    - 주 평가 지표로는 **Mind2Web**의 평가 방식을 따름
        - **step-based** attack success rate (ASR)을 사용하여 공격의 효과를 평가
        - **각 단계**에서 에이전트가, 공격자가 목표로 한 적대적 행동 a_adv = (o, r_adv, e)를 정확히 생성하면 공격이 성공한 것으로 간주됨
- **Implementation Details**
    
    - LLM 기반 공격 프롬프터
        
        → 다양한 적대적 프롬프트를 생성하여 초기 학습 데이터
        
        - **GPT-4**를 백엔드로 활용하여 프롬프트를 생성하고, 10개의 프롬프트를 생성할 때 다양성을 높이기 위해 온도를 1로 설정한다.
    - 생성 적대적 프롬프터 모델 공격자
        
        - **Mistral-7B-Instruct-v0.2**(Jiang et al., 2023)에서 초기화하며,
        - 첫 번째 SFT 단계에서는 학습률 1e-4, 배치 크기 32로 설정한다.
        - 두 번째 DPO 단계에서는 학습률을 유지하지만, 배치 크기는 16으로 줄인다.
    - SeeAct 백엔드로는 **gpt-4-vision-preview**와 **gemini-1.5-flash**를 사용한다.
        
- **Baselines**
    

현재 AdvWeb과 동일한 환경에서 웹 에이전트에 대해 블랙박스 공격을 수행하는 기존 연구가 없기 때문에, 네 가지 최신 공격 기법을 설정에 맞게 변형하여 비교 기준으로 사용한다

- **GCG** (Zou et al., 2023)
    
    기울기 기반 최적화로 목표 모델에 대한 토큰 수준의 기울기를 활용하여 **adversarial suffix string**을 최적화하는 화이트박스 공격이다. 우리의 블랙박스 설정에서는 **LLaVA-NeXT**라는 강력한 오픈소스 VLM을 대상으로 공격을 최적화한 후, 공격을 에이전트로 전이하는 방식으로 활용한다.
    
- **AutoDAN** (Liu et al., 2024c)
    
    유전자 알고리즘을 활용하여 목표 모델의 **로그(logits)**을 최적화하는 화이트박스 공격이다. 이 또한 LLaVA-NeXT에 대해 공격을 최적화한 후, 결과를 모델에 전이하는 방식으로 적용한다.
    
- **COLD-Attack** (Guo et al., 2024)
    
    에너지 기반의 **Langevin dynamics**로 제약된 디코딩을 사용하는 알고리즘이다. 기울기 기반 접근을 요구하며, 대응하는 에너지 함수를 도입하여 유창하고 스텔스성이 높은 적대적 프롬프트를 생성한다.
    
- **Catastrophic Jailbreak** (Huang et al., 2024)
    
    디코딩 방법의 변형을 통해 모델 정렬을 파괴하는 블랙박스 공격 알고리즘이다. 시스템 프롬프트를 제거하거나 디코딩 하이퍼 파라미터와 샘플링 방법을 변경하여 최소한의 계산 비용으로 모델 공격을 수행한다.
    

실험 결과

- 높은 성공률을 기록하며, 다양한 웹 도메인에서 GPT-4V 기반 SeeAct를 성공적으로 공격하는 데 있어 97.5%의 공격 성공률을 달성했다. 게다가 AdvWeb은 목표를 변경해도 추가 최적화 없이 98.5%의 성공률을 유지해 강력한 통제 가능성을 입증했다.
    
- Effectiveness of AdvWeb
    
    - **VLM-powered web agent 는 AdvWeb에 매우 취약함**
        
        - GPT-4V 백엔드를 사용하는 SeeAct에서 평균 97.5% ASR
        - Gemini 1.5 백엔드를 사용하는 SeeAct에서는 99.8% ASR
    - baselines보다 우수함
        
        기존 기법들은 화이트박스 환경에서 기울기 기반 정보를 활용하여 목표 반응을 극대화하도록 설계되었지만, **복잡한 타겟형 블랙박스 공격 시나리오에서는 ASR이 0%로 기록**되었다. ↔ AdvWeb과 대비됨
        
        ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c81e1106-6576-44de-83bf-6e68d6e7fb70/74ba6a3f-b4da-4697-82f3-786739bc0d8d/image.png)
        
- In-depth Analysis of AdvWeb
    
    - **AdvWeb은 높은 통제성을 가지고 다양한 공격 목표를 설정 가능함**
        
        - 생성된 적대적 문자열을 수정하여 새로운 목표에 맞도록 변경 평균 **98.5%의 ASR** → 통제성 입증
    - **AdvWeb은 유연하고 다양한 설정에 대한 전이성이 높음**
        
        - 성공적인 적대적 문자열을 다양한 설정으로 전이하여 평가했다. 예를 들어, 삽입 위치나 HTML 필드를 변경했을 때도 높은 성공률을 유지했으며, 위치 변화 시 **71.0%**, HTML 필드 변경 시 97.0%의 ASR을 기록했다.
    - **모델 피드백의 차이점으로부터 학습하는 것이 공격 성능 향상에 기여함**
        
        - SFT만 사용했을 때의 ASR과 SFT와 DPO를 결합한 경우를 비교한 결과, 블랙박스 모델의 피드백을 활용함으로써 공격 성공률이 크게 향상되었다. 특히, 평균 ASR이 69.5% (SFT 단독)에서 97.5% (DPO 적용 시)로 증가했다.
            
            ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c81e1106-6576-44de-83bf-6e68d6e7fb70/0b406689-f54b-4263-b993-ae7fec8d36b6/image.png)
            
    - **전이 기반 블랙박스 공격은 타겟형 공격에서 ASR이 낮음**
        
        - Gemini 1.5 백엔드를 사용하는 에이전트에 대해 GPT-4V를 대상으로 생성된 공격 문자열을 전이했을 때의 ASR이 낮았다. 이는 모델 피드백을 활용하는 블랙박스 공격이 전이 기반 알고리즘보다 성능이 우수함을 보여준다.

<case study>

미세한 차이가 있는 적대적 프롬프트가 서로 다른 공격 결과를 초래함

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c81e1106-6576-44de-83bf-6e68d6e7fb70/f83fe1e7-53f6-405b-b383-b4b2d9d0fbe1/image.png)

1. “you”를 “I”로 바꿨을 때 공격이 실패에서 성공으로 바뀌었다.
2. “previous”라는 단어를 추가하여 타겟 에이전트를 성공적으로 속일 수 있었다.

실험 결과, 이와 같은 미세한 차이들이 ASR에 큰 영향을 미쳤다. (수동으로 설계된 적대적 프롬프트 방식에서는 이러한 미세한 패턴 차이를 포착하기 어려우나, AdvWeb의 두 단계 학습 프로세스를 통해 프롬프트의 미묘한 변화를 효과적으로 학습할 수 있다.)

1. 사용자가 Microsoft 주식을 추가하도록 에이전트에 지시하지만, AdvWeb이 생성한 적대적 삽입(q) 후에 에이전트는 NVIDIA 주식을 구매하게 된다.
2. 사용자가 Tylenol의 부작용 정보를 요청하지만, 적대적 삽입 이후 에이전트는 Aspirin의 부작용을 검색하게 된다.

(AdvWeb이 적대적 공격을 통해 웹 에이전트의 동작을 조작하는 데 있어 효과적임을 잘 보여준다.)

결론

- 이번 연구의 제한사항으로는 공격 문자열 최적화가 **오프라인 피드백에 의존한다는 점**이 있다. 미래 연구에서는 블랙박스 에이전트로부터 실시간 피드백을 활용할 수 있는 적대적 프롬프터 모델을 개발하여, 공격을 실시간으로 최적화하는 방법을 탐구할 수 있다.
- 이번 연구는 웹 에이전트의 상대적으로 낮은 **end-to-end task completion rate**(작업 완료율)로 인해 step-based ASR을 주 평가 지표로 삼았다. 단계 수준의 평가가 유용한 통찰을 제공하기는 하지만, 에이전트가 전체 사용자 요청을 완료하는 과정에서의 위험을 충분히 반영하지는 못할 수 있다.

→ 실시간 인터랙티브 웹 환경 내에서 end-to-end 평가를 수행하는 방법을 고려해야 한다.

→ 작업 전체 흐름에서의 ASR을 모니터링하는 방법을 고려해야 할 것이다.

→ gpt-4-vision-preview와 gemini-1.5-flash기반 SeeAct에 현존하는 guardrail 기법을 추가