# 2. Class diagram


해당 프로젝트는 Unity를 사용하여 2D 환경의 암호 해독 시뮬레이션 게임으로 제작한다. 본 프로젝트는 플레이어가 NPC로부터 암호문과 키를 전달받고, 가이드북과 해독 기기를 이용하여 암호를 해독한 뒤 결과를 제출하거나 변조, 신고하는 흐름을 중심으로 구성된다. 따라서 클래스 구조 역시 플레이어 조작, NPC 의뢰 처리, 암호 처리, 퍼즐 UI, 정산 및 저장 시스템을 중심으로 설계하였다.

해당 클래스 다이어그램은 Project Crypto의 주요 게임 진행 흐름에 사용되는 핵심 클래스들을 표현한 다이어그램이다. 실제 구현 과정에서는 UI 버튼, 팝업창, 세부 암호 기기, 애니메이션 처리 등 더 많은 보조 클래스가 사용될 수 있으나, 본 클래스 다이어그램에서는 게임의 주요 기능을 담당하는 핵심 클래스 위주로 표현하였다.

Unity 프로젝트의 특성상 대부분의 클래스는 MonoBehaviour를 상속받아 게임 오브젝트에 컴포넌트 형태로 부착된다. MonoBehaviour는 Unity 스크립트가 기본적으로 상속받는 클래스로, Start(), Update(), 충돌 처리, 트리거 감지, 코루틴 실행 등 게임 오브젝트의 동작을 제어하는 기능을 제공한다. 본 프로젝트에서도 각 오브젝트의 역할을 독립적인 컴포넌트로 나누어 관리하며, 이를 통해 기능별 수정과 확장이 쉽도록 설계하였다.

본 프로젝트의 클래스는 크게 게임 흐름 관리, 플레이어 및 NPC 상호작용, 암호 처리 및 해독 기기, 정산 및 성장 시스템으로 구분할 수 있다.

```mermaid
classDiagram

class GameManager {
- int currentDay
- float stageTime
- bool isGameOver
+ StartGame()
+ StartStage()
+ SpawnNPC()
+ EndStage()
+ CheckGameOver()
+ CallSettlement()
+ SaveGame()
+ PlaceOwnedDevices()
}

class SaveManager {
- SaveData saveData
+ SaveFromPlayer()
+ LoadToPlayer()
+ CreateNewSave()
+ AddOwnedDevice()
+ GetOwnedDevices()
}

class SaveData {
- int day
- int money
- int reputation
- string ownedDevices
+ Serialize()
+ Deserialize()
}

class PlayerController {
- int money
- int reputation
- GameObject selectedObject
+ GetMoney()
+ AddMoney()
+ SpendMoney()
+ GetReputation()
+ AddReputation()
+ ClickObject()
+ InteractNPC()
+ OpenGuidebook()
+ SelectDevice()
+ SubmitResult()
}

class NPCController {
- RequestData currentRequest
- bool hasRequest
+ EnterShop()
+ StartDialogue()
+ GiveRequest()
+ ReceiveResult()
+ ReactToChoice()
+ ExitShop()
}

class RequestData {
- string ciphertext
- string plainText
- string key
- CipherType cipherType
- int reward
- int riskLevel
- bool canReport
+ GetCipherText()
+ GetKey()
+ GetAnswer()
}

class WorktableManager {
- GameObject cipherPaper
- GameObject keyObject
- GameObject guidebook
- GameObject reportDevice
+ PlaceRequest()
+ PlaceDevices()
+ ActivateObject()
+ ClearWorktable()
+ OpenGuidebook()
+ OpenDevice()
}

class CipherEngine {
+ Encrypt()
+ Decrypt()
+ ValidateAnswer()
}

class GuidebookManager {
- int currentPage
- CipherType selectedCipherType
+ OpenGuidebook()
+ CloseGuidebook()
+ NextPage()
+ PrevPage()
+ SelectCipherMethod()
}

class DecryptionDevice {
<>
- RequestData requestData
- string inputText
- string resultText
+ OpenPuzzle()
+ SetRequestData()
+ ApplyInput()
+ CheckResult()
+ ClosePuzzle()
}

class CaesarDevice {
- int shiftValue
+ RotateDial()
+ ConvertText()
+ CheckResult()
}

class SubstitutionDevice {
- Dictionary_char_char substitutionMap
+ SetMapping()
+ ConvertText()
+ CheckResult()
}

class ResultManager {
- string resultText
- bool isCompleted
- bool isModified
- bool isReported
+ CreateResult()
+ MarkCompleted()
+ CheckResult()
+ GetFinalResult()
+ RequestModulation()
+ RequestReport()
}

class ModulationManager {
- string originalText
- string modifiedText
+ OpenModulationUI()
+ ModifyText()
+ ApplyModifiedText()
+ SetModulationState()
}

class ReportManager {
- bool reportSuccess
+ OpenReportUI()
+ CheckReportable()
+ SendReport()
+ ApplyReportResult()
}

class SettlementSystem {
- int successCount
- int failCount
- int modulationCount
- int reportCount
- int rewardDelta
- int reputationDelta
+ CalculateReward()
+ CalculatePenalty()
+ CalculateReputation()
+ ApplyToPlayer()
+ ShowSettlement()
}

class ShopManager {
- string purchasableDevices
- string ownedDevices
+ OpenShop()
+ BuyDevice()
+ UpgradeFacility()
+ CheckMoney()
+ RequestAddOwnedDevice()
}

class CipherType {
<>
Caesar
Substitution
Playfair
Vigenere
}

GameManager --> NPCController : controls spawn
GameManager --> SettlementSystem : calls settlement
GameManager --> SaveManager : saves or loads
GameManager --> WorktableManager : places owned devices

SaveManager --> SaveData : stores snapshot
SaveManager --> PlayerController : syncs player data

PlayerController --> NPCController : interacts
PlayerController --> WorktableManager : clicks objects
PlayerController --> GuidebookManager : opens guidebook
PlayerController --> ResultManager : checks result
PlayerController --> NPCController : delivers result

NPCController --> RequestData : creates request

RequestData --> CipherType : has type
RequestData --> WorktableManager : used for placement

WorktableManager --> RequestData : places request objects
WorktableManager --> GuidebookManager : opens guidebook
WorktableManager --> DecryptionDevice : manages devices

GuidebookManager --> CipherType : selects method
GuidebookManager --> DecryptionDevice : opens matched device

DecryptionDevice --> RequestData : uses request data
DecryptionDevice --> CipherEngine : sends input

DecryptionDevice <|-- CaesarDevice
DecryptionDevice <|-- SubstitutionDevice

CaesarDevice --> CipherEngine : checks caesar result
SubstitutionDevice --> CipherEngine : checks substitution result

ResultManager --> CipherEngine : validates result
ResultManager --> ModulationManager : requests modulation
ResultManager --> ReportManager : requests report

SettlementSystem --> ResultManager : reads result records
SettlementSystem --> PlayerController : applies money and reputation
SettlementSystem --> ShopManager : enables upgrade phase

ShopManager --> PlayerController : checks and spends money
ShopManager --> SaveManager : updates owned devices
ShopManager --> DecryptionDevice : unlocks devices
```

## 2.1 Class diagram
### Game System
| 클래스명   | 주요 역할  | 주요 기능 |
| :------------------- | :---------------------------- | :------------------------------------------------- |
| **GameManager** | 게임의 전체 흐름을 관리하는 최상위 클래스  | 스테이지 시작, NPC 생성, 영업 시간 관리, 하루 종료, 게임 오버 및 엔딩 조건 관리 |
| **SaveManager**  | 게임 진행 상황 저장 및 불러오기 담당 | 자금, 평판, 날짜, 보유 기기, 진행 상태 저장 및 |

### Player / NPC
| 클래스명   | 주요 역할  | 주요 기능 |
| :------------------- | :---------------------------- | :------------------------------------------------- |
| **PlayerController**     | 플레이어의 입력과 행동을 관리 | 마우스 클릭, UI 선택, 오브젝트 상호작용, 자금 및 평판 정보 관리  |
| **NPCController**     | 게임 진행 상황 저장 및 불러오기 담당         | 자금, 평판, 날짜, 보유 기기, 진행 상태 저장 및 |
### Request / Worktable
| 클래스명   | 주요 역할  | 주요 기능 |
| :------------------- | :---------------------------- | :------------------------------------------------- |
| **RequestData**  | NPC가 전달하는 의뢰 정보를 저장하는 데이터 클래스 | 암호문, 원본 평문, 키 값, 암호 방식, 보상, 위험도, 신고 가능 여부 저장       |
| **WorktableManager**  |  작업대 위 오브젝트의 배치와 상호작용 관리  | 암호문 서류, 키 아이템, 가이드북, 해독 기기, 전화기 활성화 및 제어  |
### Cipher / Decryption
| 클래스명   | 주요 역할  | 주요 기능 |
| :------------------- | :---------------------------- | :------------------------------------------------- |
| **CipherEngine**        | 암호화 및 복호화 검증을 담당하는 핵심 클래스     | 평문을 암호문으로 변환, 플레이어 입력값과 정답 비교, 해독 결과 검증            |
| **GuidebookManager**        | 암호 해독 규칙을 확인하는 가이드북 UI 관리     | 페이지 넘김, 목차 이동, 암호 방식 선택, 해독 기기 연결    |
| **DecryptionDevice**| 해독 기기들의 부모 클래스       | 퍼즐 UI 열기, 키 값 설정, 입력값 반영, 결과 확인 요청   |
| **CaesarDevice** |시저 암호 방식의 해독 기기 클래스           | 키 값에 따른 문자 이동, 암호문을 평문 후보로 변환       |
| **SubstitutionDevice** |단순 치환 암호 방식의 해독 기기 클래스        | 암호 문자와 평문 문자의 대응 관계 설정, 치환 결과 생성         |
### Result / Choice
| 클래스명   | 주요 역할  | 주요 기능 |
| :------------------- | :---------------------------- | :------------------------------------------------- |
| **ResultManager**  | 독 완료 후 생성된 평문 문서 관리  | 평문 생성, 제출 가능 상태 변경, 변조 여부 표시, 신고 가능 여부 확인  |
| **ModulationManager**   | 해독된 평문 변조 기능 관리    | 평문 일부 수정, 거짓 문장 교체, 변조 상태 기록   |
| **ReportManager** |  해독 결과 신고 기능 관리 | 전화기 또는 신고 UI를 통한 신고 처리, 보상 및 평판 변화 반영 |
### Settlement / Upgrade
| 클래스명   | 주요 역할  | 주요 기능 |
| :------------------- | :---------------------------- | :------------------------------------------------- |
| **SettlementSystem**  | 하루 종료 시 성과 정산 담당   | 해독 건수, 오답, 변조, 신고 보상, 세금, 이자 계산 |

## 2.2 Class Relationship Summary

| 연결 관계  | 설명   |
| :---------------------------------------- | :------------------------------------------------------------------------------------- |
| `GameManager` → `NPCController` | 스테이지가 시작되면 `GameManager`가 NPC 등장을 제어하고, `NPCController`가 NPC의 이동과 대화를 처리한다. |
| `NPCController` → `RequestData` | NPC는 의뢰에 필요한 암호문, 키, 보상, 암호 방식 등의 정보를 담은 `RequestData`를 생성하거나 전달한다. |
| `RequestData` → `WorktableManager` | `WorktableManager`는 전달받은 의뢰 정보를 바탕으로 작업대 위에 암호문 서류와 키 아이템을 배치한다. |
| `PlayerController` → `WorktableManager` | 플레이어가 작업대 위 오브젝트를 클릭하면 `PlayerController`가 해당 입력을 감지하고 `WorktableManager`에 상호작용을 요청한다. |
| `PlayerController` → `GuidebookManager` | 플레이어가 가이드북을 클릭하면 `GuidebookManager`가 가이드북 UI를 열고 암호 해독 규칙을 보여준다. |
| `GuidebookManager` → `DecryptionDevice`  | 플레이어가 해독 방법을 선택하면 해당 암호 방식에 맞는 해독 기기 또는 퍼즐 UI가 실행된다. |
| `DecryptionDevice` → `CipherEngine` | 해독 기기에서 입력된 값은 `CipherEngine`으로 전달되어 정답 여부를 검증한다. |
| `CaesarDevice` → `DecryptionDevice`| `CaesarDevice`는 `DecryptionDevice`를 상속받아 시저 암호에 맞는 문자 이동 조작을 구현한다. |
| `SubstitutionDevice` → `DecryptionDevice` | `SubstitutionDevice`는 `DecryptionDevice`를 상속받아 단순 치환 암호의 문자 대응 조작을 구현한다. |
| `CipherEngine` → `ResultManager` | 해독 결과가 올바르면 `ResultManager`가 평문 문서를 생성하고 제출 가능한 상태로 변경한다. |
| `ResultManager` → `NPCController` | 플레이어가 평문을 제출하면 NPC가 결과물을 받고 의뢰를 종료한다. |
| `ResultManager` → `ModulationManager` | 플레이어가 변조를 선택하면 `ModulationManager`가 평문 내용을 수정하고 변조 상태를 기록한다.  |
| `ResultManager` → `ReportManager` | 플레이어가 신고를 선택하면 `ReportManager`가 평문 내용을 신고 대상으로 전달한다. |
| `GameManager` → `SettlementSystem` | 하루가 종료되면 `GameManager`가 `SettlementSystem`을 호출하여 일일 정산을 진행한다.  |
| `SettlementSystem` → `ShopManager` | 정산이 끝나면 플레이어는 획득한 자금을 사용해 `ShopManager`에서 시설을 업그레이드할 수 있다. |
| `GameManager` → `SaveManager` | 하루 종료 또는 수동 저장 시 `GameManager`가 `SaveManager`를 호출하여 현재 진행 상황을 저장한다. |

## 2.3 Class Group Summary

| 그룹 | 포함 클래스   | 설명      |
| :--------- | :------------------------------------------------------------------------------------------- | :------------------------------------------ |
| 게임 흐름 관리   | `GameManager`, `SaveManager`  | 게임의 시작, 진행, 저장, 종료를 관리하는 시스템 클래스  |
| 플레이어 및 NPC | `PlayerController`, `NPCController`| 플레이어 입력과 NPC 의뢰 흐름을 담당하는 클래스  |
| 의뢰 및 작업대   | `RequestData`, `WorktableManager`| NPC가 전달한 의뢰 정보와 작업대 오브젝트 배치를 관리하는 클래스  |
| 암호 처리 및 해독 | `CipherEngine`, `GuidebookManager`, `DecryptionDevice`, `CaesarDevice`, `SubstitutionDevice` | 암호문 생성, 해독 규칙 확인, 퍼즐 조작, 해독 결과 검증을 담당하는 클래스 |
| 결과 처리  | `ResultManager`, `ModulationManager`,`ReportManager` | 평문 제출, 변조, 신고 등 플레이어의 선택 결과를 처리하는 클래스  |
| 정산 및 성장 | `SettlementSystem`, `ShopManager`                                                            | 하루 결과를 계산하고, 자금을 이용한 시설 업그레이드를 처리하는 클래스     |


## 2.4 Overall Flow

|  순서 | 동작 흐름 | 관련 클래스 |
| :-: | :------------------------------- | :------------------------------------------------------- |
|  1  | 게임 또는 스테이지가 시작 | `GameManager` |
|  2  | NPC가 상점에 등장하고 플레이어와 대화 | `NPCController`, `PlayerController` |
|  3  | NPC가 암호문과 키가 포함된 의뢰를 전달 | `NPCController`, `RequestData` |
|  4  | 작업대 위에 암호문 서류와 키 아이템이 배치 | `WorktableManager` |
|  5  | 플레이어가 가이드북을 열어 해독 규칙을 확인 | `PlayerController`, `GuidebookManager` |
|  6  | 플레이어가 해독 기기를 선택하고 퍼즐을 진행 | `DecryptionDevice`, `CaesarDevice`, `SubstitutionDevice` |
|  7  | 입력된 해독 결과를 검증 | `CipherEngine`  |
|  8  | 올바른 해독 결과가 나오면 평문 문서가 생성 | `ResultManager`   |
|  9  | 플레이어는 평문 제출, 변조, 신고 중 하나를 선택  | `ResultManager`, `ModulationManager`, `ReportManager` |
|  10 | 하루가 종료되면 의뢰 결과를 정산 | `SettlementSystem`  |
|  11 | 정산 후 시설 업그레이드 또는 해독 기기 구매를 진행 | `ShopManager`  |
|  12 | 현재 진행 상황을 저장   | `SaveManager`      |