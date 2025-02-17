# C++课程设计：拼图游戏

## 1. 实验内容

玩家可以通过鼠标单击相邻的空位方块，使其移动，最终将被打乱的图片恢复成完整图案。游戏的具体要求如下：

1.  **游戏启动与初始化**
    -   游戏启动后，显示初始界面（图一）。
    -   玩家按下空格键进入游戏，系统将完整图片分割为 15 个大小相等的方块，并随机打乱摆放，形成游戏初始状态（图二）。
2.  **游戏操作**
    -   进入游戏后，玩家可以通过鼠标单击相邻的空位方块，使其移动到空位处。
    -   只有与空位相邻的方块可以移动，玩家需逐步调整方块位置，最终拼出完整的图案。
    -   当方块排列恢复至目标状态（右下角留空），游戏判定胜利，并返回初始界面（图一）。
3.  **游戏原理**
    -   采用 `4×4` 的矩阵（二维数组）来表示游戏方块位置。
    -   前 15 个方块的编号按顺序为 `1~15`，第 16 个格子为空（编号 `0`）。
    -   游戏初始化时，系统会随机打乱这些编号，并确保至少存在一种可解路径。
    -   `0` 代表空位，只有与 `0` 相邻的方块可以移动至空位。
4.  **交互规则**
    -   玩家单击鼠标左键即可移动方块，每次只能移动一格。
    -   只有当鼠标位于可移动方块的范围内，点击才会生效。

## 2. 实验指南

### 2.1 开始实验

### 2.2 单击空格键，开始游戏

#### 【实验内容】

启动游戏时，屏幕上显示“空格开始”提示，玩家按下空格键后，进入游戏初始界面，并隐藏提示文字。

#### 【实验思路】

- 系统会自动调用 `OnKeyDown` 函数响应键盘按下事件。
- 代码实现位于 `main.cpp`，需要在 `OnKeyDown` 函数内检测空格键的按下状态。
- 当空格键被按下时，修改 `m_iGameState` 变量，使游戏进入开始状态，并隐藏“空格开始”精灵。

#### 【实验指导】

##### 1. 添加变量声明

在 `LessonX.h` 文件中，声明“空格开始”精灵变量，并定义 `OnKeyDown` 函数：

```cpp
CSprite* m_spGameBegin; // "空格开始"精灵

void OnKeyDown(const int iKey, const int iAltPress, const int iShiftPress, const int iCtrlPress);
```

##### 2. 初始化“空格开始”精灵

在 `LessonX.cpp` 的构造函数中初始化 `m_spGameBegin` 变量：

```cpp
m_spGameBegin = new CSprite("GameBegin");
```

在 `LessonX.cpp` 末尾添加 `OnKeyDown` 函数的定义：

```cpp
void CGameMain::OnKeyDown(const int iKey, const int iAltPress, const int iShiftPress, const int iCtrlPress) {
}
```

##### 3. 处理空格键事件

- 空格键的键值为 `KEY_SPACE`。
- 仅当 `m_iGameState == 0` 时，按下空格键才会触发游戏开始。
- 在 `OnKeyDown` 函数中添加以下代码：

```cpp
if (iKey == KEY_SPACE && m_iGameState == 0) {
    m_iGameState = 1; // 进入游戏状态
    m_spGameBegin->SetSpriteVisible(false); // 隐藏 "空格开始" 精灵
}
```

##### 4. 绑定键盘输入处理

在 `Main.cpp` 文件中，将系统的按键输入事件传递到 `OnKeyDown` 进行处理。

在 `OnKeyDown` 调用处添加以下代码：

```cpp
g_GameMain.OnKeyDown(iKey, bAltPress, bShiftPress, bCtrlPress);
```

#### 【实验总结】

本实验通过监听空格键的输入，实现了游戏的启动机制。按下空格键后，游戏状态改变，并隐藏“空格开始”提示，为后续的游戏逻辑奠定了基础。


### 2.3 初始化随机显示方块

#### 【实验内容】

1. 创建一个 4x4 的二维数组 `m_iBlockState`，用于存储 15 个方块的位置，剩下一个空位用于移动方块。
2. 使用 `iRandData` 数组存储 1-15 的数值，并随机排列方块。
3. 确保每个方块在初始化时不会重复。

#### 【实验思路】

- 在 4x4 矩阵 `m_iBlockState` 中，前 15 个值存储 1-15，最后一个位置留空（标记为 0）。
- 初始化时，从 `iRandData` 数组中随机选择一个值，赋给 `m_iBlockState`，并创建对应的精灵。
- 使用 `m_spBlock` 数组存储 `CSprite` 对象，并将每个精灵放置到相应的位置。
- 通过 `XYToOneIndex` 进行索引转换，使一维数组 `m_spBlock` 和二维数组 `m_iBlockState` 之间能正确映射。

#### 【实验指导】

##### 1. 添加成员变量声明

在 `LessonX.h` 头文件中，声明以下变量：

```cpp
static const float m_fBlockStartX;
static const float m_fBlockStartY;
static const float m_fBlockSize;
int m_iBlockState[BLOCK_COUNT][BLOCK_COUNT];
CSprite* m_spBlock[BLOCK_COUNT * BLOCK_COUNT];
```

添加 `BLOCK_COUNT` 宏定义：

```cpp
#define BLOCK_COUNT 4 // 4x4 矩阵方块
```

##### 2. 在 `LessonX.cpp` 定义常量变量

```cpp
const float CGameMain::m_fBlockSize = 18.75f;
const float CGameMain::m_fBlockStartX = -40.625f;
const float CGameMain::m_fBlockStartY = -28.125f;
```

##### 3. 在 `GameInit` 中初始化方块数据

```cpp
int iLoopX = 0, iLoopY = 0, iLoop = 0;
int iOneIndex = 0, iRandIndex = 0;
int iDataCount = BLOCK_COUNT * BLOCK_COUNT - 1;
int iRandData[BLOCK_COUNT * BLOCK_COUNT - 1] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15};
```

##### 4. 定义索引转换函数

在 `LessonX.h` 中声明：

```cpp
int XYToOneIndex(const int iIndexX, const int iIndexY);
```

在 `LessonX.cpp` 实现：

```cpp
int CGameMain::XYToOneIndex(const int iIndexX, const int iIndexY) {
    return (iIndexY * BLOCK_COUNT + iIndexX);
}
```

##### 5. 遍历二维数组并随机初始化方块

```cpp
for (iLoopY = 0; iLoopY < BLOCK_COUNT; iLoopY++) {
    for (iLoopX = 0; iLoopX < BLOCK_COUNT; iLoopX++) {
        iOneIndex = XYToOneIndex(iLoopX, iLoopY);
        
        // 设定空位
        if (iLoopX == BLOCK_COUNT - 1 && iLoopY == BLOCK_COUNT - 1) {
            m_iBlockState[iLoopY][iLoopX] = 0;
            m_spBlock[iOneIndex] = new CSprite("NULL");
        } else {
            // 随机选择一个未使用的数值
            iRandIndex = CSystem::RandomRange(0, iDataCount - 1);
            m_iBlockState[iLoopY][iLoopX] = iRandData[iRandIndex];
            char* tmpName = CSystem::MakeSpriteName("PictureBlock", m_iBlockState[iLoopY][iLoopX]);
            m_spBlock[iOneIndex] = new CSprite(tmpName);
            MoveSpriteToBlock(m_spBlock[iOneIndex], iLoopX, iLoopY);
            
            // 移动剩余数据，防止重复
            for (iLoop = iRandIndex; iLoop < iDataCount - 1; iLoop++) {
                iRandData[iLoop] = iRandData[iLoop + 1];
            }
            iDataCount--;
        }
    }
}
```

##### 6. 定义移动精灵到指定位置的函数

在 `LessonX.h` 中声明：

```cpp
void MoveSpriteToBlock(CSprite *tmpSprite, const int iIndexX, const int iIndexY);
```

在 `LessonX.cpp` 中实现：

```cpp
void CGameMain::MoveSpriteToBlock(CSprite *tmpSprite, const int iIndexX, const int iIndexY) {
   float fPosX = m_fBlockStartX + iIndexX * m_fBlockSize;
   float fPosY = m_fBlockStartY + iIndexY * m_fBlockSize;
   tmpSprite->SetSpritePosition(fPosX, fPosY);
}
```

#### 【实验总结】

本实验实现了 4x4 方块矩阵的随机初始化，并确保每个方块不会重复。通过索引转换，我们能够在一维和二维数组之间高效操作，使初始化更高效可靠。

### 2.4 移动方块

#### 【实验内容】

1. 获取鼠标单击消息。
2. 判断鼠标点击的方块。
3. 判断周围是否有空位，并移动方块。

#### 【实验思路】

- 遍历一维数组 `m_spBlock`，使用 `IsPointInSprite` 判断鼠标点击是否位于某个精灵内部。
- 如果找到点击的精灵，将索引赋值给 `iClickIndex`。
- 检查该方块周围是否有名称为“NULL”的精灵，如果有，则移动至该位置。

#### 【实验指导】

##### 1. 添加鼠标点击处理函数声明

在 `LessonX.h` 中添加 `OnMouseClick` 函数声明：

```cpp
void OnMouseClick(const int iMouseType, const float fMouseX, const float fMouseY);
```

##### 2. 添加 `OnMouseClick` 函数定义

在 `LessonX.cpp` 中实现 `OnMouseClick`：

```cpp
void CGameMain::OnMouseClick(const int iMouseType, const float fMouseX, const float fMouseY) {
    // 只处理游戏进行中的鼠标响应
    if (m_iGameState != 2) return;

    int iClickIndex = -1;

    for (int iLoop = 0; iLoop < BLOCK_COUNT * BLOCK_COUNT; iLoop++) {
        if (m_spBlock[iLoop]->GetName() == "NULL") continue;

        if (m_spBlock[iLoop]->IsPointInSprite(fMouseX, fMouseY)) {
            iClickIndex = iLoop;
            break;
        }
    }

    if (iClickIndex == -1) return;

    int iIndexX = OneIndexToX(iClickIndex);
    int iIndexY = OneIndexToY(iClickIndex);
    int iEmptyIndexX = -1, iEmptyIndexY = -1;

    // 判断四个方向是否有空位
    if (iIndexX > 0 && m_iBlockState[iIndexY][iIndexX - 1] == 0) {
        iEmptyIndexX = iIndexX - 1;
        iEmptyIndexY = iIndexY;
    }
    else if (iIndexX < BLOCK_COUNT - 1 && m_iBlockState[iIndexY][iIndexX + 1] == 0) {
        iEmptyIndexX = iIndexX + 1;
        iEmptyIndexY = iIndexY;
    }
    else if (iIndexY > 0 && m_iBlockState[iIndexY - 1][iIndexX] == 0) {
        iEmptyIndexX = iIndexX;
        iEmptyIndexY = iIndexY - 1;
    }
    else if (iIndexY < BLOCK_COUNT - 1 && m_iBlockState[iIndexY + 1][iIndexX] == 0) {
        iEmptyIndexX = iIndexX;
        iEmptyIndexY = iIndexY + 1;
    }

    if (iEmptyIndexX == -1 || iEmptyIndexY == -1) return;

    // 交换方块位置（不使用 std::swap）
    int tempState = m_iBlockState[iIndexY][iIndexX];
    m_iBlockState[iIndexY][iIndexX] = m_iBlockState[iEmptyIndexY][iEmptyIndexX];
    m_iBlockState[iEmptyIndexY][iEmptyIndexX] = tempState;

    int iOneIndex = XYToOneIndex(iEmptyIndexX, iEmptyIndexY);
    CBlock* tempBlock = m_spBlock[iClickIndex];
    m_spBlock[iClickIndex] = m_spBlock[iOneIndex];
    m_spBlock[iOneIndex] = tempBlock;

    // 移动方块至新位置
    MoveSpriteToBlock(m_spBlock[iOneIndex], iEmptyIndexX, iEmptyIndexY);
}
```

##### 3. 添加辅助函数

在 `LessonX.h` 中声明：

```cpp
int OneIndexToX(const int iIndex);
int OneIndexToY(const int iIndex);
```

在 `LessonX.cpp` 中实现：

```cpp
int CGameMain::OneIndexToX(const int iIndex) {
    return iIndex % BLOCK_COUNT;
}

int CGameMain::OneIndexToY(const int iIndex) {
    return iIndex / BLOCK_COUNT;
}
```

##### 4. 在 `Main.cpp` 处理鼠标点击事件

```cpp
g_GameMain.OnMouseClick(iMouseType, fMouseX, fMouseY);
```

#### 【实验总结】

本实验实现了鼠标点击方块的检测，并判断周围是否有空位来移动方块。通过索引转换，我们能够在一维数组和二维数组之间进行操作，使方块能够正确移动到空位上。

### 2.5 判断游戏是否胜利

#### 【实验内容】

1. 循环遍历判断是否胜利。
2. 胜利后重新开始游戏。

#### 【实验思路】

判断游戏是否胜利的核心逻辑是：
- 所有方块按照 1 至 15 的顺序排列，并且第 16 个位置的值为 0。
- 该判断函数需要在程序每次调用主循环时检测一次。

#### 【实验指导】

##### 1. 添加 IsGameWin 函数声明

在 `LessonX.h` 中添加自定义的判断是否胜利的函数 `IsGameWin` 声明：

```cpp
int IsGameWin();
```

##### 2. 添加 IsGameWin 函数定义

在 `LessonX.cpp` 中，添加 `IsGameWin` 函数定义。在文件末尾添加如下代码：

```cpp
int CGameMain::IsGameWin() {
    // 待实现逻辑
}
```

##### 3. 实现游戏胜利判断逻辑

- 使用两个 `for` 循环遍历二维数组 `m_iBlockState`。
- 检查前 15 个元素是否依次递增（即 `1, 2, ..., 15`）。
- 第 16 个元素应为 0。
- 如果遍历过程中发现任意一个元素不符合条件，立即返回 `0`（未胜利）。
- 若所有条件满足，则返回 `1`（胜利）。

在 `IsGameWin` 函数内添加如下代码：

```cpp
int iLoopX = 0, iLoopY = 0;
int iResult = 1;

for (iLoopY = 0; iLoopY < BLOCK_COUNT; iLoopY++) {
    for (iLoopX = 0; iLoopX < BLOCK_COUNT; iLoopX++) {
        // 判断是否为最后一个位置
        if (iLoopX == BLOCK_COUNT - 1 && iLoopY == BLOCK_COUNT - 1) {
            return (m_iBlockState[iLoopY][iLoopX] == 0) ? 1 : 0;
        }
        
        // 若值不符合预期，则游戏未胜利
        if (m_iBlockState[iLoopY][iLoopX] != iResult) {
            return 0;
        }
        
        iResult++;
    }
}

return 1;
```

##### 4. 在主循环中检测游戏胜利状态

- 需要定期检测游戏状态，在 `GameMainLoop` 中修改 `GameRun` 调用。
- 原代码：

  ```cpp
  if (true) {
      GameRun();
  }
  ```

- 修改为：

  ```cpp
  if (!IsGameWin()) {
      GameRun();
  }
  ```

##### 5. 处理游戏胜利后的逻辑

- 如果游戏胜利，则执行 `GameEnd` 函数。
- 在 `GameEnd` 函数中添加以下代码，以显示游戏开始提示：

```cpp
// 显示提示开始文字
m_spGameBegin->SetSpriteVisible(1);
```

#### 【实验总结】

本实验实现了一个判断游戏胜利的函数，并在主循环中调用该函数以实时检测游戏状态。如果检测到胜利，则执行相应的游戏结束逻辑并重新开始游戏。



