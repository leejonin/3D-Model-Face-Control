# 3D-Model-Face-Control

**KR**: Unity에서 LLM(대화 모델)이 출력한 **이모지**로 3D 캐릭터의 **표정(BlendShape)** 과 일부 이펙트를 제어하는 샘플입니다.  
**EN**: A Unity sample where an LLM controls a 3D character’s **facial expressions (BlendShapes)** and some effects using **emoji output**.

---

## 한국어 (KR)

### 제작/출처
- **Unity**: 6000.0.35f1
- **모델 제작**: VRoid Hub (`https://hub.vroid.com/en`)에서 모델 생성 후 Unity로 임포트

### 포함 파일
- `Code/FaceReact.cs`: 이모지 → 표정/이펙트 매핑, `FaceMove(string input)`로 이모지 입력 처리
- `Code/LipSync.cs`: 한글 모음 기반 간단 립싱크(오디오 재생 시간에 맞춰 입 BlendShape 가동)
- `Date/unicode_emoji_sample.json`: 이모지 데이터 샘플(JSON)

> 중요: 이 레포에서는 이모지 JSON을 **프로젝트 루트의 `Date/unicode_emoji_sample.json`**에서 읽는 것을 기준으로 합니다.

### Unity 씬 세팅(Inspector 드래그&드롭)

`FaceReact` 컴포넌트를 캐릭터(또는 매니저 오브젝트)에 붙인 뒤 아래를 연결합니다.

1) **얼굴 BlendShape 렌더러 연결**
- `public SkinnedMeshRenderer face;`
  - 캐릭터의 얼굴(BlendShape가 들어있는) **Skinned Mesh Renderer**를 드래그&드롭

2) **립싱크 연결**
- `public LipSync lip;`
  - 같은 오브젝트(또는 씬 어딘가)에 `LipSync` 컴포넌트를 추가하고, 그 컴포넌트를 드래그&드롭  
  - 그리고 `LipSync` 안의 `public FaceReact face;`에도 `FaceReact`를 연결(상호 참조)

3) **볼 이펙트(Ohg)**
- `public GameObject Ohg;`
  - 씬에 `Hongo` 오브젝트를 넣고 캐릭터 볼(뺨) 쪽에 배치한 뒤 **비활성화(Off)** 해둡니다.
  - 그 오브젝트를 `Ohg`에 드래그&드롭

4) **하트 프리팹(heart)**
- `public GameObject heart;`
  - `Heart` 프리팹을 `heart`에 드래그&드롭

5) **하트 목표 위치(t_heart)**
- `public GameObject t_heart;`
  - 하트가 “날아가서 도착할” 위치에 빈 GameObject를 씬에 하나 생성 후,
  - 그 오브젝트를 `t_heart`에 드래그&드롭

> 참고: 현재 `FaceReact.cs`의 하트 이동 코루틴은 **고정 좌표**(`new Vector3(0.0608f, 1.431f, -0.1668f)`)로 호출됩니다.  
> `t_heart`를 실제로 쓰는 구조로 바꾸고 싶다면, `MoveToPosition(t_heart.transform.position)`로 연결하도록 코드를 조정하면 됩니다.

### LLM 연동(핵심)

#### 1) System 프롬프트에 JSON 전달 + “이모지=얼굴 제어용” 명시
LLM에 API Key로 요청할 때, **system** 메시지에 `unicode_emoji_sample.json` 내용을 포함시키고 아래 문장을 명시하세요.
- **“이 JSON의 이모지들은 얼굴 제어용이다.”**

예시(System 메시지 개념):
- 역할: system
- 내용:
  - “너는 감정/표정을 이모지로만 출력한다.”
  - “아래 JSON에 있는 이모지들은 얼굴 제어용이다. 출력은 JSON에 있는 이모지 중 하나 이상만 사용해라.”
  - (여기에 `Date/unicode_emoji_sample.json` 내용을 그대로 첨부)

#### 2) LLM 출력값을 `FaceMove`에 넣기
LLM의 최종 출력(예: `"😂"` 또는 `"😁😂"` 같은 문자열)을 그대로 아래처럼 전달합니다.

```csharp
face.FaceMove(LLM_output);
```

`FaceMove(string input)`은 문자열을 “텍스트 요소(이모지 포함)” 단위로 순회하며, JSON에 등록된 이모지와 매칭되면 표정 코루틴을 실행합니다.

---

## English (EN)

### Built with / Source
- **Unity**: 6000.0.35f1
- **Model**: Created via VRoid Hub (`https://hub.vroid.com/en`) and imported into Unity

### Included files
- `Code/FaceReact.cs`: Emoji → facial expression/effects mapping. Call `FaceMove(string input)` with emoji text.
- `Code/LipSync.cs`: Simple Korean-vowel-based lip sync driven by audio clip timing.
- `Date/unicode_emoji_sample.json`: Sample emoji mapping JSON.

> Important: This repo assumes the emoji JSON is loaded from **`Date/unicode_emoji_sample.json`** at the project root.

### Unity scene setup (Inspector drag & drop)

Attach `FaceReact` to your character (or a manager object), then wire these fields:

1) **Face BlendShape renderer**
- `public SkinnedMeshRenderer face;`
  - Drag the character’s face **Skinned Mesh Renderer** (the one containing BlendShapes).

2) **Lip sync**
- `public LipSync lip;`
  - Add `LipSync` to an object, then drag that component into `lip`.
  - In `LipSync`, also set `public FaceReact face;` to point back to this `FaceReact`.

3) **Cheek effect (Ohg)**
- `public GameObject Ohg;`
  - Place a `Hongo` object near the character’s cheek in the scene, keep it **disabled** by default,
  - Then drag it into `Ohg`.

4) **Heart prefab**
- `public GameObject heart;`
  - Drag the `Heart` prefab into `heart`.

5) **Heart target position**
- `public GameObject t_heart;`
  - Create an empty GameObject at the desired “fly-to” destination,
  - Drag it into `t_heart`.

> Note: The current code calls the heart movement with a **hard-coded position** (`new Vector3(0.0608f, 1.431f, -0.1668f)`).  
> If you want to use `t_heart`, update the call to `MoveToPosition(t_heart.transform.position)`.

### LLM integration (core)

#### 1) Provide the JSON in the system prompt + state it’s for face control
When calling the LLM with your API key, include the JSON content in the **system** message and explicitly state:
- **“These emojis are for facial control.”**

Conceptual system prompt:
- “You output emojis only.”
- “The emojis in the JSON below are for facial control. Output must be one or more emojis from that JSON.”
- Paste the content of `Date/unicode_emoji_sample.json` into the system message.

#### 2) Feed LLM output into `FaceMove`
Pass the LLM’s final emoji output (e.g., `"😂"` or `"😁😂"`) directly:

```csharp
face.FaceMove(LLM_output);
```

`FaceMove(string input)` iterates text elements (including emojis) and triggers the mapped BlendShape/effect routines.

