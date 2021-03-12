#
```
教科書
Python 網路爬蟲與資料分析入門實戰
繁體中文/林俊瑋、林修博/博碩文化出版日期：2018-10-04
https://github.com/jwlin/py-scraping-analysis-book
ch2
```
# 學習主題
```
主題1:搜尋技術
主題2:網頁深層結構解析技術
主題3:正規表達法的技術
```
# 主題1:搜尋技術
```
import requests
from bs4 import BeautifulSoup


def main():
    resp = requests.get('http://jwlin.github.io/py-scraping-analysis-book/ch2/blog/blog.html')
    soup = BeautifulSoup(resp.text, 'html.parser')

    # 取得第一篇 blog (h4)
    print(soup.find('h4'))
    print(soup.h4)  # 與上一行相等

    # 取得第一篇 blog 主標題
    print(soup.h4.a.text)

    # 取得所有 blog 主標題, 使用 tag
    main_titles = soup.find_all('h4')
    for title in main_titles:
        print(title.a.text)

    # 取得所有 blog 主標題, 使用 class
    # 以下寫法皆相同:
    # soup.find_all('h4', 'card-title')
    # soup.find_all('h4', {'class': 'card-title'})
    # soup.find_all('h4', class_='card-title')
    main_titles = soup.find_all('h4', 'card-title')
    for title in main_titles:
        print(title.a.text)

    # 使用 key=value 取得元件
    print(soup.find(id='mac-p'))

    # 當 key 含特殊字元時, 使用 dict 取得元件
    # print(soup.find(data-foo='mac-foo'))  # 會導致 SyntaxError
    print(soup.find('', {'data-foo': 'mac-foo'}))

    # 取得各篇 blog 的所有文字
    divs = soup.find_all('div', 'content')
    for div in divs:
        # 方法一, 使用 text (會包含許多換行符號)
        #print(div.text)
        # 方法二, 使用 tag 定位
        #print(div.h6.text.strip(), div.h4.a.text.strip(), div.p.text.strip())
        # 方法三, 使用 .stripped_strings
        print([s for s in div.stripped_strings])


if __name__ == '__main__':
    main()
```

# 主題2:網頁深層結構解析技術
```
import requests
from bs4 import BeautifulSoup


def main():
    resp = requests.get('http://jwlin.github.io/py-scraping-analysis-book/ch2/table/table.html')
    soup = BeautifulSoup(resp.text, 'html.parser')

    # 計算課程均價
    # 取得所有課程價錢: 方法一, 使用 index
    prices = []
    rows = soup.find('table', 'table').tbody.find_all('tr')
    for row in rows:
        price = row.find_all('td')[2].text
        prices.append(int(price))
    print(sum(prices)/len(prices))

    # 取得所有課程價錢: 方法二, <a> 的 parent (<td>) 的 previous_sibling
    prices = []
    links = soup.find_all('a')
    for link in links:
        price = link.parent.previous_sibling.text
        prices.append(int(price))
    print(sum(prices) / len(prices))

    # 取得每一列所有欄位資訊: find_all('td') or row.children
    rows = soup.find('table', 'table').tbody.find_all('tr')
    for row in rows:
        all_tds = row.find_all('td')  # 方法一: find_all('td')
        # all_tds = [td for td in row.children]  # 方法二: 找出 row (tr) 所有的直接 (下一層) children
        # 以下執行時會報錯, 因為最後一列的 <a> 沒有 'href' 屬性
        # print(all_tds[0].text, all_tds[1].text, all_tds[2].text, all_tds[3].a['href'], all_tds[3].a.img['src'])
        # 取得 href 屬性前先檢查其是否存在
        if 'href' in all_tds[3].a.attrs:
            href = all_tds[3].a['href']
        else:
            href = None
        print(all_tds[0].text, all_tds[1].text, all_tds[2].text, href, all_tds[3].a.img['src'])

    # 取得每一列所有欄位文字資訊: stripped_strings
    rows = soup.find('table', 'table').tbody.find_all('tr')
    for row in rows:
        print([s for s in row.stripped_strings])


if __name__ == '__main__':
    main()
```
# 主題3:正規表達法的技術
```
import requests
import re
from bs4 import BeautifulSoup


def main():
    resp = requests.get('http://jwlin.github.io/py-scraping-analysis-book/ch2/blog/blog.html')
    soup = BeautifulSoup(resp.text, 'html.parser')

    # 找出所有 'h' 開頭的標題文字
    titles = soup.find_all(['h1', 'h2', 'h3', 'h4', 'h5', 'h6'])
    for title in titles:
        print(title.text.strip())

    # 利用 regex 找出所有 'h' 開頭的標題文字
    for title in soup.find_all(re.compile('h[1-6]')):
        print(title.text.strip())

    # 找出所有 .png 結尾的圖片
    imgs = soup.find_all('img')
    for img in imgs:
        if 'src' in img.attrs:
            if img['src'].endswith('.png'):
                print(img['src'])

    # 利用 regex 找出所有 .png 結尾的圖片
    for img in soup.find_all('img', {'src': re.compile('\.png$')}):
        print(img['src'])

    # 找出所有 .png 結尾且含 'beginner' 的圖片
    imgs = soup.find_all('img')
    for img in imgs:
        if 'src' in img.attrs:
            if 'beginner' in img['src'] and img['src'].endswith('.png'):
                print(img['src'])

    # 利用 regex 找出所有 .png 結尾且含 'beginner' 的圖片
    for img in soup.find_all('img', {'src': re.compile('beginner.*\.png$')}):
        print(img['src'])


if __name__ == '__main__':
    main()
```
