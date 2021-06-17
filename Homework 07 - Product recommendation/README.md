## Product recommendation:

Data source:  https://github.com/sukitpom/BADS7105/blob/master/Homework%2007%20-%20Product%20recommendation/Customer%20Preference%20Survey.csv

## Exploratory Data Analysis:
![data prepare](https://github.com/sukitpom/BADS7105/blob/master/Homework%2007%20-%20Product%20recommendation/img/prepare_data.png)

![Top 10](https://github.com/sukitpom/BADS7105/blob/master/Homework%2007%20-%20Product%20recommendation/img/Top_10.png)

![Top 10](https://github.com/sukitpom/BADS7105/blob/master/Homework%2007%20-%20Product%20recommendation/img/Last.png)

## Collaborative Filtering:(item-based)

![cosine similarity](https://github.com/sukitpom/BADS7105/blob/master/Homework%2007%20-%20Product%20recommendation/img/cosine_similarity.png)

### cosine similarity> 0.5
มีสินค้าจำนวนหนึ่งเกาะกลุ่ม และมีบางส่วนแยกออกมาจากกลุ่ม
![match 0.5](https://github.com/sukitpom/BADS7105/blob/master/Homework%2007%20-%20Product%20recommendation/img/recom_050.png)

### cosine similarity> 0.7
![match 0.7](https://github.com/sukitpom/BADS7105/blob/master/Homework%2007%20-%20Product%20recommendation/img/recom_070.png)

### cosine similarity> 0.95
จะเริ่มเห็นสินค้าที่จับกลุ่มกันมาก เพราะว่าเป็นสินค้าที่มีความสนใจใน user ค่อนข้างมาก
![match 0.95](https://github.com/sukitpom/BADS7105/blob/master/Homework%2007%20-%20Product%20recommendation/img/recom_095.png)

### network circle graph
ลองแสดงผลในลักษณะ circle
![Circle](https://github.com/sukitpom/BADS7105/blob/master/Homework%2007%20-%20Product%20recommendation/img/circular.png)
