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

## フィールド情報
- 途中に謎のパディング？があり開始アドレスはヘッダから次のバイト数を読み飛ばした先
  - `((width - 1) / 2 + 1) * ((height - 1) / 4 + 1) * 2`

```
for y=0 to height - 1
  for x=0 to width - 1
    data[x,y].worldIndex[0].w
    data[x,y].worldIndex[1].w
    data[x,y].worldIndex[2].w
    data[x,y].flag.l
```

- flag & 1: 歩行可否

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

## オブジェクト情報
フィールド情報の直後から

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

## その他謎情報
オブジェクトの続き
```
count1.l
for i=0 to count1 - 1
  unk.8b

count2.l
for i=0 to count2 - 1
  unk.b
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
