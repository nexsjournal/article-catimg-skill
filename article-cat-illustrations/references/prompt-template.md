# 生图提示词模板

每张图单独生成，通过 qiyuan-image 技能调用 qwen-image API。根据正文内容替换变量，不要把多张图拼在一起。

## 文生图（默认路径）

```bash
node ~/.codex/skills/qiyuan-image/qiyuan-image.mjs generate \
  --prompt "见下方模板" \
  --out assets/<article-slug>-illustrations/0N-topic-name.png \
  --size 1664x928
```

```text
Generate one standalone 16:9 horizontal Chinese article illustration.

Visual DNA:
Pure white background. Minimalist hand-drawn line art. Slightly wobbly pen lines. Lots of empty white space. Sparse red/orange/blue handwritten Chinese annotations. Clean absurd product-sketch feeling. No gradients, no shadows, no paper texture, no complex background, no commercial vector style, no PPT infographic look, no cute mascot poster, no children's illustration, no realistic UI.

Recurring IP character required:
A tortoiseshell (three-color) cat: black, ginger/tan and white fur patches; a white blaze running down the nose and a white muzzle; lime-green round eyes; pointed dark ears; short white paws. Deadpan, aloof, slightly grumpy expression, like a serious system operator who happens to be a cat. The cat must perform the core conceptual action, not decorate the scene. Keep the cat serious and deadpan, NOT cute, NOT chibi, NOT sparkly.

Theme:
{正文配图主题}

Structure type:
{结构类型：Workflow / 系统局部 / 前后对比 / 角色状态 / 概念隐喻 / 方法分层 / 地图路线 / 小漫画分镜}

Core idea:
{这张图要表达的核心意思}

Composition:
{具体画面：猫在哪里、正在做什么、主要物件是什么、信息如何流动}

Suggested elements:
{元素1} / {元素2} / {元素3} / {元素4}

Chinese handwritten labels:
{标注词1} / {标注词2} / {标注词3} / {标注词4} / {可选标注词5}

Color use:
Black, ginger and white for the cat. Black for main line art and structure. Orange for main flow/path/arrows. Red only for key warnings/problems/results. Blue only for secondary notes or feedback/system state.

Constraints:
One image explains only one core structure. Keep the main subject around 40%-60% of the canvas. Preserve at least 35% blank white space. Use at most 5-8 short handwritten Chinese labels. Do not write a title in the top-left corner. Do not write the structure type on the image. Do not make it a formal diagram, course slide, or dense explainer. Do not copy prior examples or reuse known case compositions unless explicitly requested; invent a fresh visual metaphor for this specific article. It should be clear but not instructional, interesting but not childish, strange but clean.
```

注意：当前 qiyuan-image CLI 的 generate 子命令传 `--negative` 会报错（form.append bug），请勿在 generate 中传该参数；negative 内容已并入提示词。

```text
cute, chibi, kawaii, mascot, sparkle eyes, cartoon baby face, bow, collar, ribbon, PPT, infographic, 3D render, photo, gradient, shadow, paper texture
```

## 角色一致性 / 局部编辑（edit 命令）

猫的毛色或表情偏离参考猫，或需要局部修改时，用真实猫照做参考图：

```bash
node ~/.codex/skills/qiyuan-image/qiyuan-image.mjs edit \
  --image <skill-dir>/assets/cat/cat-01.png \
  --prompt "保持参考图中这只三花猫（黑橘白三色毛、浅绿圆眼、冷淡脸）的外貌特征完全一致，把它放进目标构图：<目标构图描述>。纯白背景，黑色手绘线稿风格，大量留白，少量红橙蓝中文手写批注，16:9 横版。" \
  --out assets/<article-slug>-illustrations/0N-topic-name.png \
  --strength 0.6
```

去掉左上角标题：

```text
Edit the provided image. Remove only the handwritten title "{要删除的文字}" and its underline from the top-left corner. Fill that area with the same clean white background, matching the surrounding blank paper. Preserve everything else exactly: the cat, labels, paths, line style, composition, aspect ratio, and image quality. Do not add any new text or objects.
```

增强角色参与感：

```text
Regenerate this illustration with the same core meaning and simple layout, but make the tortoiseshell cat more central to the conceptual action. The cat should be doing the strange work that explains the idea, not standing beside the diagram. Keep it clean, sparse, hand-drawn, and not cute.
```

## strength 调参经验

- 0.4-0.5：构图基本跟随提示词，猫只取参考图的部分特征。
- 0.6：默认起点，猫特征与构图平衡。
- 0.7-0.85：猫非常接近参考图，但构图会被参考图带偏。

## 实测调参经验（qwen-image）

- 提示词过长/结构太复杂（Theme/Structure type/Core idea 分节 + 大量英文约束）时，模型容易画成写实素描或照片，且会幻觉出大段中文。**收敛成一段自然语言描述 + 明确 "hand-drawn ink sketch on pure white paper, not a photo, no camera frame, no dark edges"**，扁平手绘感会稳定很多。
- 中文标注必须写"exactly ONE annotation, do not repeat"，否则模型会把同一句话在每个元素旁重复多遍。
- 纯白背景要在提示词里强调 "entire canvas edge to edge pure white, no vignette, no room, no photo"，否则画面右侧/边缘常出现暗色"照片边"。
- 猫的长相（三花毛色、绿眼、冷淡脸）模型还原度很高，文生图即可；edit 参考图主要用来修局部，不必作为默认路径。
