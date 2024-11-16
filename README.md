# bilibili-dynamic-exiftool-download-by-image-url
查看b站动态exif


```
!apt install exiftool
import requests
import subprocess
import re
from bs4 import BeautifulSoup

class EXIFExtractor:
    def __init__(self):
        self.ignore = [
            "忽略掉的exif行",
            "File Permissions",
            "File Type Extension",
            "Profile Description",
            "Encoding Process",
            "X Resolution",
            "Y Resolution",
            "JFIF Version",
            "Resolution Unit",
            "Displayed Units"
        ]
        self.user_agent = 'Mozilla/5.0 (compatible; MSIE 9.0; Windows Phone OS 7.5; Trident/5.0; IEMobile/9.0; HTC; Titan)'

    def exif(self, url):
        headers = {
            'User-Agent': self.user_agent
        }

        response = requests.get(url, headers=headers, allow_redirects=True)
        final_url = response.url
        #print(response.text)
        html_content = response.text
        html_content = html_content.replace("\u002F", "/")
        soup = BeautifulSoup(html_content, 'html.parser')
        decoded_text = bytes(response.text, "utf-8").decode("unicode-escape")
        clean_pattern = re.compile('<.*?>')
        clean_text = re.sub(clean_pattern, '', decoded_text)
        pattern = r'(https?://i0.hdslb.com/bfs/new_dyn/[^\s"]+)'
        matches = re.findall(pattern, clean_text)
        photo_number = 1

        for match in matches:
            print(f'提取的内容: 图{photo_number}, ： {match}')
            img_url = match
            img_path = 'downloaded_image.jpg'
            curl_result = subprocess.run(['curl', '-o', img_path, img_url], stdout=subprocess.PIPE)

            if curl_result.returncode == 0:
                exiftool_result = subprocess.run(['exiftool', '-G', img_path], stdout=subprocess.PIPE, text=True)
                exiftool_output = exiftool_result.stdout
            else:
                exiftool_output = "curl command failed"

            exiftool_lines = exiftool_output.split('\n')
            filtered_lines = [line for line in exiftool_lines if not any(keyword in line for keyword in self.ignore)]
            
            for a in filtered_lines:
                print(a)
            print()
            photo_number += 1

# 示例调用
extractor = EXIFExtractor()
extractor.exif('https://b23.tv/E1q35L9')
```
