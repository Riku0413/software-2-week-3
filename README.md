# software-2-week-3

## 課題1

### 変更点1
長方形と円の描画のコマンドを識別する値を列挙型に追加。
```
typedef enum res{ EXIT, LINE, RECT, CIRCLE, UNDO, SAVE, UNKNOWN, ERRNONINT, ERRLACKARGS, NOCOMMAND} Result;
```

### 変更点2
長方形と円を描く関数のプロトタイプ宣言。
```
void rect(Canvas *c, const int x0, const int y0, const int width, const int height);
void circle(Canvas *c, const int x0, const int y0, const int r);
```

### 変更点3
長方形、円を描画したときも履歴に追加する。
```
if (r == LINE | r == RECT | r == CIRCLE) {
		// [*]
		push_command(&his,buf);
}
```

### 変更点4
実際に長方形と円を描画する関数。
```
void rect(Canvas *c, const int x0, const int y0, const int width, const int height)
{
    char pen = c->pen;

    if ( (x0 >= 0) && (x0 < c->width) && (y0 >= 0) && (y0 < c->height))
	c->canvas[x0][y0] = pen;
    for (int i = 1; i <= width; i++) {
	const int x = x0 + i;
	const int y = y0;
	if ( (x >= 0) && (x < c->width) && (y >= 0) && (y < c->height))
	    c->canvas[x][y] = pen;
    }
    for (int i = 1; i <= width; i++) {
	const int x = x0 + i;
	const int y = y0 + height;
	if ( (x >= 0) && (x < c->width) && (y >= 0) && (y < c->height))
	    c->canvas[x][y] = pen;
    }
    for (int i = 1; i <= height; i++) {
	const int x = x0;
	const int y = y0 + i;
	if ( (x >= 0) && (x < c->width) && (y >= 0) && (y < c->height))
	    c->canvas[x][y] = pen;
    }
    for (int i = 1; i <= height; i++) {
	const int x = x0 + width;
	const int y = y0 + i;
	if ( (x >= 0) && (x < c->width) && (y >= 0) && (y < c->height))
	    c->canvas[x][y] = pen;
    }

}

double distance(int x0, int y0, int x1, int y1) {
    double d = sqrt((x0-x1)*(x0-x1) + (y0-y1)*(y0-y1));
    return d;
}

void circle(Canvas *c, const int x0, const int y0, const int r)
{
    char pen = c->pen;

    for (int x = 0; x < c->width; x++) {
        for (int y = 0; y < c->height; y++) {
            if (distance(x0, y0, x, y) > r - 0.5 && distance(x0, y0, x, y) < r + 0.5) {
                c->canvas[x][y] = pen;
            }
        }
    }
}
```

### 変更点5
長方形と円の描画のコマンドを解釈するプログラム。
```
    if (strcmp(s, "rect") == 0) {
	int p[4] = {0}; // p[0]: x0, p[1]: y0, p[2]: width, p[3]: height
	char *b[4];
	for (int i = 0 ; i < 4; i++){
	    b[i] = strtok(NULL, " ");
	    if (b[i] == NULL){
		return ERRLACKARGS;
	    }
	}
	for (int i = 0 ; i < 4 ; i++){
	    char *e;
	    long v = strtol(b[i],&e, 10);
	    if (*e != '\0'){
		return ERRNONINT;
	    }
	    p[i] = (int)v;
	}

	rect(c,p[0],p[1],p[2],p[3]);
	return RECT;
    }

    if (strcmp(s, "circle") == 0) {
	int p[3] = {0}; // p[0]: x0, p[1]: y0, p[2]: r
	char *b[3];
	for (int i = 0 ; i < 3; i++){
	    b[i] = strtok(NULL, " ");
	    if (b[i] == NULL){
		return ERRLACKARGS;
	    }
	}
	for (int i = 0 ; i < 3 ; i++){
	    char *e;
	    long v = strtol(b[i],&e, 10);
	    if (*e != '\0'){
		return ERRNONINT;
	    }
	    p[i] = (int)v;
	}

	circle(c,p[0],p[1],p[2]);
	return CIRCLE;
    }
```

### 変更点6
長方形と円の描画時の表示文を設定。
```
  case RECT:
return "1 rect drawn";
  case CIRCLE:
return "1 circle drawn";
```

## 課題2

### 変更点1
下記のようにloadコマンドが成功した時の識別子LOADと失敗した時のERRORを追加。
```
typedef enum res{ EXIT, LINE, RECT, CIRCLE, LOAD, ERROR, UNDO, SAVE, UNKNOWN, ERRNONINT, ERRLACKARGS, NOCOMMAND} Result;
```

### 変更点2
load()関数のプロトタイプ宣言。
```
int load(Canvas *c, History *his, const char *txt_filename);
```

### 変更点3
load関数の中身を定義。
参照するテキストファイルの中身を1行ずつ読み込み、各行にinterpret_command()関数とpush_command()関数を適用するという形で実装している。
```
int load(Canvas *c, History *his, const char *txt_filename)
{
// 1行ずつテキストを読み込む
    FILE *fp;
		if ((fp = fopen(txt_filename, "r")) == NULL) {
			return 0;
    }
// 各行に対して, interpret_commandを実行
		char str[100];
		while (fgets(str, 100, fp) != NULL) {
			interpret_command(str, his, c); // 怪しい
			push_command(his, str);
			// for (int i = 0; i < 100; i++) {
			// 	str[i] = 0;
			// }
		}
// 行が全部終わったら, 終了 
    fclose(fp);
		return 1;
}
```

### 変更点4
interpret_command()関数内にloadコマンドを読み取るプログラムを追加。
```
if (strcmp(s, "load") == 0) {
	char *b[1];
	b[0] = strtok(NULL, " "); // 引数のファイル名を読み取り
	if (b[0] == NULL) {
		b[0] = "history.txt";
	}
	int r = load(c, his, b[0]);
	if (r == 0) {
		return ERROR;
	} else {
		return LOAD;
	}
}
```

### 変更点5
loadコマンド実行時の表示文を追加。
```
	case LOAD:
return "1 text data reproduced";
	case ERROR:
return "error: cannot open file";
```

## 課題3