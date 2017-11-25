# YPF
## Header
```
unknown.w //旧framecountとのコメントあり
width.w
height.w
depthRes.l
paletteFlag.l (bool)

if paletteFlag
  paletteEntry.w * 256

dataPos.l //データ部の開始アドレス
imageCount.w //ImageSet

for imageCount
  unknown1.l
  if unknown1
    unknown2.l
  
  frameCount.w
  for frameCount
    top.w
    left.w
    bottom.w
    right.w
    flag.l
    
    if flag & 8 == 0 // hasAlpha
      length.l
      dataOffset.l
      
    if flag & 4 // hasBase
      dataOffset.l
      length.l
      
    if flag & 1 // hasDepth
      val1.b
      val2.w
      val3.w
      dataOffset.l
      length.l
      
    unk2Length.l
    if unk2Length
      unk2offset.l
      
  animationCount.w
  for animationCount
    //unknown.16b
    anim1Size.l
    if anim1Size > 0
      anim1Offset.l
    
    for 3
      unk1.b
      unk2.b
      unk3.b
      unk4.b
    
    segmentCount.w
    for segmentCount
      segIndex.w
      time.l
      length.l
      if length
        dataOffset.l
  
  stateCount.w
  for stateCount
    stateSize.l
    if stateSize
      stateOffset.l
    stateElemCount.w
    for stateElemCount
      isStateElemFrame.b
      unk1.b
      unk2.b
      unk3.w
      if isStateElemFrame
        unk4.w
      stateElemSize.l
      if stateElemSize
        stateElemOffset.l
    
  stateTransValueCount.w
  for stateTransValueCount
    key1.w
    key2.w
    value.w
    stateTransValueSize.l
    if stateTransValueSize
      stateTransValueOffset.l
```

## Base

### Palette
paletteFlag>0のときだけ
```
paletteIndex.b * (width * height)
```

## Alpha
- alphaデータのオフセット位置から
- 行ごとに圧縮されている

### フォーマット
```
rowSize1.l
for height
  rowSize2.l

for width * height
  data.w //0x7fffして555
```

### 展開
```
rowSize = getLong();
for(int i=0; i<height; i++) {
  rowSize2 = getLong();
  rows[i] = (rowSize2 - rowSize) / 2;
  rowSize = rowSize2;
}

for(int y=0; y<height; y++) {
  x = 0;
  for(int i=0; rows[y]<width; i++) {
    data1 = getByte();
    data2 = getByte();
    for(int j=0; j<((data1 & 7) << 8 | data2); j++) {
      data[x++ + y * width] = data1 & 0xf8;
    }
  }
}

return data;
```

## Depth
1/depthResな解像度の深度情報
```
data.w * (ceil(width / depthRes) * ceil(height / depthRes))
```

## Animation
### Segment
dataOffsetからの相対
```
offsetX.b
offsetY.b
unknown.w
```

## StateTransValue
key1, key2はペアでキーとして利用される
```cpp
std::hashmap< std::pair<short, short>, StateTransValue> stateTransValues;
stateTransValues.insert(std::make_pair(key1, key2), stv);
```

# DOI
Draw Order Information?か何かだと思う。

複数パーツを持つオブジェクトの描画順を定義するもの。
今のところHuman系でしか見てない

```
unknown.w //使ってなさそう
count.w
for i=0 to count - 1
  data.b
```

## データ形式
dataはframeのindex番目を参照する
* 6～7ビット目: 体の描画順
* 4～5ビット目: 頭の描画順
* 2～3ビット目: 武器の描画順
* 0～1ビット目: シールドの描画順

```
body = (data >> 6) & 3
hair = (data >> 4) & 3
weapon = (data >> 2) & 3
shield = (data >> 0) & 3
```

ここで得られた順番で各要素を描画する。

それぞれ2, 3, 0, 1だったら武器、シールド、体、頭の順で描画
