# MapFile
## Header
```
unknown.l //1固定？
worldId[0].w
worldId[1].w
worldId[2].w
width.b
height.b
```

## MapAnim
* ヘッダの直後から
* 中身はこれから追う
```
for y=0 to (height - 1) / 4
  for x=0 to (width - 1) / 2
    unk.b
    unk.b
```

## MapTile
* MapAnimの直後から
```
for y=0 to height - 1
  for x=0 to width - 1
    data[x,y].worldIndex[0].w
    data[x,y].worldIndex[1].w
    data[x,y].worldIndex[2].w
    data[x,y].flag.l
```

- タイルごとの参照すべきworld*.datの番号。タイルは最大3枚重なる
- flag & 1: 歩行可否
- flag & 0x10: SpellVisibility // 調査中

### 描画
ピクセルの合成方法はypf内で指定されているもの
```
for y=0 to height - 1
  for x=0 to width - 1
    for i=0 to 2
      ypf = open ("TS_" + worldId[i] + "_Tile.ypf")
      frame = ypf.image[0].frame[ worldIndex[i] ]
      draw (frame.image, (x * 48) + frame.bounds.x, (y * 32) + frame.bounds.y) // 48,32 はtileの固定サイズ
```

## StaticObjectInfo
MapTileの直後から

```
count.l
for i=0 to count - 1
  obj[i].isAnim.b
  obj[i].fileIndex.w
  obj[i].frameIndex.w
  obj[i].x.w
  obj[i].y.w
```
- isAnim: アニメーションオブジェクトかどうか
  - true: `"TS_" + fileIndex + "_AnimStatic(Shadows)?.ypf"`なypfを描画する
  - false: `"TS_" + fileIndex + "_Static(Shadows)?.ypf"`なypfを描画する
- fileIndex: ypfのファイル名インデックス
- frameIndex: ypfImage内のframeインデックス
- x, y: 描画する位置(絶対座標 in pixel)。ここにframe.boundsがオフセットされる

## LightInfo
StaticObjectInfoの続き
```
count1.l
for i=0 to count1 - 1
  unk.w
  x.w
  y.w
  unk.w
```

## MultiStateObjectInfo
LightInfoの続き
```
count2.l
for i=0 to count2 - 1
  unk.w
  unk.w
  unk.l
  alpha.b //色はどこかで逆になったかも…
  blue.b
  green.b
  red.b
  unk.w
  unk.w
```

## ポータル
\#basic.datのほうに書くべきか
portals.tbl
```
unk.b //1?
count.l
for i=0 to count - 1
  mapId.w
  x.w
  y.w // height超えていないかだけチェックしてる
```
