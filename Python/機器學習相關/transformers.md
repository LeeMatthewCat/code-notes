來自 Hugging Face 的套件

# 導入

## ACT2FN 激活函數映射字典

Activation to（2，諧音）Function，

將激活函數的名稱映射到 PyTorch 的激活函數物件

本質上就是有一個名為 ACT2FN 的字典，裡面包含這鍵（激活函數字串）和值（在 PyTorch 裡的實際函數）

```Python
config 中...

hidden_act = "silu"
---------------------
使用時：

... = ACT2FN[args.hidden_act] # 由 config 中的字串去 ACT2FN 字典中找值
```

> 字典中的內容：
> 
> ```Python
> ACT2CLS = {
>     "gelu": GELUActivation,
>     "gelu_10": (ClippedGELUActivation, {"min": -10, "max": 10}),
>     "gelu_fast": FastGELUActivation,
>     "gelu_new": NewGELUActivation,
>     "gelu_python": (GELUActivation, {"use_gelu_python": True}),
>     "gelu_pytorch_tanh": GELUTanh,
>     "gelu_python_tanh": (GELUTanh, {"use_gelu_tanh_python": True}),
>     "gelu_accurate": AccurateGELUActivation,
>     "hardswish": nn.Hardswish,
>     "laplace": LaplaceActivation,
>     "leaky_relu": nn.LeakyReLU,
>     "linear": LinearActivation,
>     "mish": MishActivation,
>     "quick_gelu": QuickGELUActivation,
>     "relu": nn.ReLU,
>     "relu2": ReLUSquaredActivation,
>     "relu6": nn.ReLU6,
>     "sigmoid": nn.Sigmoid,
>     "silu": SiLUActivation,
>     "sqrtsoftplus": SqrtSoftplusActivation,
>     "swish": nn.SiLU,
>     "tanh": nn.Tanh,
>     "prelu": nn.PReLU,
>     "xielu": XIELUActivation,
> }
> ACT2FN = ClassInstantier(ACT2CLS)
> ```

---

## AutoTokenizer\.from\_pretrained 載入 tokenizer

```Python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("線上名稱或本地路徑")
```

```Python
... = tokenizer 物件(
    text=資料集,
    text_pair="資料集 2", # 如果是問答型的資料集上面放問題，這裡放回答
    padding="max_length", # 是否填補，Ture：補到該批次最長長度，false 預設
    truncation=True, # 是否截斷（超出最長的被切斷丟掉）
    max_length=16, # 最大長度
    return_tensors="pt", # 回傳型別，預設 List，pt 為 PyTorch 張量
    add_special_tokens=True # 是否自動生成標籤
)
```

的到的資料形式**長得像字**典，裡面包著串列

```Python
{
    'input_ids': [1, 342, 1105, 233, 2],
    'attention_mask': [1, 1, 1, 1, 1] # 遮罩，1 表示有東西， 0 表示是填充
}

... = token.input_ids # 可以直接拿出來（）
```

---

### 標籤

附屬在 tokenizer 的屬性

```Python
xxx.bos_token # 文字形式

xxx.bos_token_id # 數字形式
```

- **\<s\>，bos**：**B**eginning of Sentence，開頭

- **\</s\>，eos**：End of Sentence，結尾

- **pad**：填充

- **unk**：未知，如果是詞彙庫裡沒有的詞

---

### apply\_chat\_template\(\) 多倫對話統一格式

- 包含了把** BOS、EOS 加入對話起點及終點**

- 直接轉 token（可選）

```Python
... = tokenizer 物件.apply_chat_template(
    conversation=多倫對話內容, # 接受串列
    tokenize=True, # 是否迴轉 tokenize 結果
    add_generation_prompt=False, # 是否在最後面加入 assistant\n
    tools=工具列表 # 沒有就傳入 None（預設）
)
```

```Python
... = [
    {"role": "system", "content": "你是一個幽默的 AI 助手。"},
    {"role": "user", "content": "請告訴我一個冷笑話。"},
    {"role": "assistant", "content": "為什麼皮卡丘走路很慢？因為他是「卡」丘。"},
    {"role": "user", "content": "好冷，再來一個。"}
]
```

---

# 配置

## AutoConfig 配置及初始化模型

用於自動讀取並實體化對應的配置類別

可以修改已有模型的參數，以可從頭開始配置

包含以下兩步驟：

1. 定義結構（Config） 

2. 初始化模型 

### Configuration **定義配置**

#### AutoConfig\.from\_pretrained 導入現有結構

```Python
config = AutoConfig.from_pretrained("現有模型")
```

```Python
from transformers import AutoConfig, AutoModelForCausalLM

# 1. 載入官方設定作為基底
config = AutoConfig.from_pretrained("Qwen/Qwen2-0.5B")

# 2. 修改特定參數把它縮小 (注意：不要去動 vocab_size)
config.num_hidden_layers = 4 # 層數
config.hidden_size = 256  # 隱藏層維度 
config.num_attention_heads = 4 # 多頭注意力層數 Multi-Head Attention
config.intermediate_size = 1024# MLP 內部 feed 的層數
```







---

# 管理

## PreTrainedModel 類別

是所有 **Hugging Face 模型的基底**，它不負責任何數學運算，而是負責所有**工程管理**的瑣事

繼承後模型擁有：

1. **from\_pretrained\(\)：**下載權重、解析 config 檔、參數填入

2. **save\_pretrained\(\)**：權重儲存

3. 自動切片（分給多個 GPU 使用）

而此類別也繼承自 **nn\.module**，繼承自 PreTrainedModel 的類別要**進行 forward 函數**的實作

---

### config\_class 登記設定檔

當繼承了 PreTrainedModel，必須在類別內宣告這個屬性

```Python
class ...(PreTrainedModel, ...):
    config_class = 設定檔類別
```

---

### from\_pretrained\(\) 權重載入

包含兩個重要屬性要在**模型檔**建立好：

1. **model**：串接拼裝  **Embedding \-\> Transformer Layer \(GQA \+ FFN\) \-\> Norm** 的類別

2. **lm\_head**：線性層，返回 logits（未經 softmax 的機率）

```Python
class ...(PreTrainedModel, ...):
    def __init__(self, ...):
        self.model = 拼裝類別
        self.lm_head = 線性層實作
```







---

### save\_pretrained\(\)







---

## CausalLMOutputWithPast 打包前向傳播

前向傳播**終點的打包**

提供**屬性存取**，用 output\.loss 或 output\.logits 拿取張量

```Python
output = CausalLMOutputWithPast(
    loss=..., # 零維純量
    logits=..., # (batch_size, seq_len, vocab_size)
    past_key_values=..., # Tuple[Tuple[torch.Tensor]]
    hidden_states=..., # Tuple[torch.Tensor]，選填
    attentions=..., # 每一層的權重，Tuple[torch.Tensor]，選填
)
```

> 通常在因果語言模型中會被打包在 模型名ForCausalLM 類別裡，而產生出模型物件
> 
> 透過呼叫此模型物件**得出結果物件後可呼叫得到上述屬性**

---

# 生成

## GenerationMixin 類別

整體來說就是一個迴圈，把預測出來的字再拼回輸入，重新丟回給模型，不段重複（接龍）直到結束符號出現

### generate\(\)

#### prepare\_inputs\_for\_generation









---

