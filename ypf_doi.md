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
  
  unknownDataCount.w
  for unknownDataCount
    val1.l
    val2.w
    flag.b
    val3.b
    val4.b
    main.w
    if flag
      flagVal.l
    
    unkSize.l
    if unkSize
      unkOffset.l
    
  unknownData2Count.w
  for unknownData2Count
    index.w
    val2.w
    val3.w
    unkSize.l
    if unkSize
      unkOffset.l
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
