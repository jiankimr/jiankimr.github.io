EMBODIED AGENT INTERFACE(EAI)는 Large Language Models(LLMs)를 활용한 Embodied Decision-Making 성능을 체계적으로 평가하고, 다양한 작업에서 LLM의 강점과 약점을 진단하기 위해 설계된 평가 프레임워크이다. EAI는 선형시간논리(LTL; Linear Temporal Logic)를 기반으로 목표를 정의하고, 네 가지 주요 능력 모듈(Goal Interpretation, Subgoal Decomposition, Action Sequencing, Transition Modeling)을 통해 작업의 복잡한 요소를 표준화하며, 세분화된 평가 지표를 제공한다.

### **1. 연구 배경**

1. **LLM과 Embodied Decision-Making의 연관성**  
    LLM은 자연어 목표를 이해하고 다양한 디지털 및 물리적 환경에서 이를 달성할 수 있는 강력한 도구로 발전하였다.  
    예: "냉장고를 청소하라"는 지시에 따라 필요한 행동을 순차적으로 수행하여 목표를 달성.
    
2. **기존 연구의 한계**  
    기존 Embodied Decision-Making 연구는 다음과 같은 문제를 가진다:
    
    - **목표 정의의 비표준화**: 작업별로 목표를 정의하는 방식과 평가 기준이 크게 다르다.
    - **모듈별 성능 비교 어려움**: 서로 다른 LLM 기반 프레임워크에서 입력-출력 사양이 상이하다.
    - **세분화된 평가 부족**: 단순한 성공률에 의존하여 세부적인 오류 분석이 어렵다.
3. **EAI의 필요성**  
    이러한 한계를 해결하기 위해 EAI는 통합된 평가 프레임워크를 제공하며, 목표 달성을 위한 세분화된 분석을 가능하게 한다.
    

### **2. EAI의 주요 구성 요소**

1. **목표 정의의 표준화 (Standardization of Goal Specification)**  
    EAI는 목표를 LTL 공식을 사용해 정의한다. LTL은 상태 기반 목표(state-based goals)와 시간적 순서가 요구되는 목표(temporally extended goals)를 모두 포괄한다.
    
    - 예: `ontop(chicken, plate) ∧ ontop(plate, table)`  
        이는 닭고기를 접시에 올리고, 접시를 테이블 위에 올리는 작업을 명시한다.
2. **모듈식 설계 (Modularized Design)**  
    EAI는 Embodied Decision-Making을 네 가지 능력 모듈로 분해한다:
    
    - **Goal Interpretation**: 자연어 목표를 LTL 형식으로 변환한다.
    - **Subgoal Decomposition**: 최종 목표를 달성하기 위한 하위 목표를 생성한다.
    - **Action Sequencing**: 하위 목표를 달성하기 위한 행동 시퀀스를 생성한다.
    - **Transition Modeling**: 행동의 전제 조건과 결과를 정의한다.
3. **세분화된 평가 지표 (Fine-Grained Metrics)**
    
    - 오류를 세부적으로 분류하여 분석한다.
    - 주요 오류 유형:
        - **환각 오류(Hallucination)**: 존재하지 않는 객체나 상태를 생성.
        - **누락 오류(Missing Goals)**: 목표의 일부를 예측하지 못함.
        - **추가 단계 오류(Additional Steps)**: 불필요한 행동을 포함.
### **3. EAI의 설계와 구현**

1. **객체 중심 모델링 (Object-Centric Modeling)**  
    EAI는 상태를 `(U, F)` 형태로 모델링한다.
    
    - `U`: 객체의 집합 (예: 냉장고, 천, 접시).
    - `F`: 객체 간의 관계 및 상태를 나타내는 Boolean 특성.
2. **LTL 기반 목표 정의 (LTL-Based Goal Representation)**  
    선형시간논리를 사용해 목표를 정의하며, 상태와 행동을 논리적으로 표현한다.
    
    - **상태 정의**: `stained(fridge)` → 냉장고가 더럽다.
    - **행동 정의**: `CLEAN(fridge)` → 냉장고를 청소한다.
3. **평가 벤치마크 (Evaluation Benchmarks)**  
    EAI는 두 가지 주요 벤치마크(BEHAVIOR, VirtualHome)에서 실험적으로 검증된다.
    
    - **BEHAVIOR**: 상태 기반 목표와 복잡한 전이 모델을 포함.
    - **VirtualHome**: 객체 간 관계와 시간적 순서를 중시.

---

### **4. 주요 능력 모듈 설명**

1. **목표 해석 (Goal Interpretation)**
    
    - **기능**: 자연어로 주어진 목표를 환경의 상태 및 객체로 변환한다.
    - **예시**: "물을 마셔라" → `open(freezer)`와 같은 중간 목표 포함.
2. **하위 목표 분해 (Subgoal Decomposition)**
    
    - **기능**: 최종 목표를 달성하기 위한 하위 목표를 생성한다.
    - **예시**: `CLEAN(fridge)`를 위해 `SOAK(rag)`, `GRASP(rag)`와 같은 단계적 작업을 생성.
3. **행동 시퀀싱 (Action Sequencing)**
    
    - **기능**: 실행 가능한 행동 시퀀스를 생성하여 시뮬레이터에서 실행 가능성을 평가한다.
    - **예시**:
        
        `RIGHT_GRASP(rag.0) CLEAN(fridge.97)`
        
4. **전이 모델링 (Transition Modeling)**
    
    - **기능**: 행동의 전제 조건과 결과를 정의하며, 논리적 구조를 명확히 기술한다.
    - **예시**:
        
        `:action soak :parameters (?rag - object, ?sink - sink) :precondition (toggled_on(?sink)) :effect (soaked(?rag))`

### **5. 실험 결과**

1. **모델별 성능 비교**
    
    - `o1-preview`, `Claude-3.5 Sonnet`, `Gemini 1.5 Pro`가 가장 높은 성능을 기록했다.
    - BEHAVIOR에서는 계획의 복잡성과 긴 행동 시퀀스로 인해 실행 성공률이 낮았다.
    - VirtualHome에서는 더 다양한 객체 상태를 다루지만 작업이 상대적으로 간단했다.
2. **오류 분석**
    
    - **목표 해석 오류**: LLM은 중간 목표를 최종 목표로 오해하거나, 특정 관계를 누락하는 경우가 많았다.
    - **행동 시퀀싱 오류**: 추가 단계 오류와 누락 단계 오류가 빈번했다.
    - **전이 모델링 오류**: 논리적 전이와 객체 관계를 정확히 이해하지 못하는 경우가 있었다.

### **6. 결론**

1. **기여**
    
    - EAI는 Embodied Decision-Making을 위한 LLM의 성능을 체계적으로 평가하는 데 필수적인 도구를 제공한다.
    - 표준화된 목표 정의와 모듈화된 평가 프레임워크를 통해 LLM의 강점과 약점을 명확히 분석한다.
2. **향후 연구 방향**
    
    - Vision-Language Models(VLM)와의 통합을 통해 시각적 입력과 행동 계획의 연계를 강화한다.
    - 복잡한 환경에서의 공간적 추론과 네비게이션 능력을 개선한다.
    - 기억 시스템(episodic memory, state memory)을 추가하여 장기적인 계획 능력을 지원한다.