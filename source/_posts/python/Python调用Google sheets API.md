---
title: Python调用Google sheets API  
date: 2017-07-20 08:44:09  
tags: [python]  
categories: python
---
##### 获取认证json文件
1. 打开Google API控制台网页([wizard](https://console.developers.google.com/flows/enableapi?apiid=sheets.googleapis.com))，点击继续，然后跳转到凭据页面。  
2. 在添加凭据页面点击取消按钮。  
3. 选择OAuth consent screen，填写邮箱和产品名称后保存。  
<!-- more -->
4. 选择Credentials，点击创建凭据按钮，选择OAuth client ID。  
5. 选择应用类型Other，填写名称点击创建。  
6. 点击OK后下载json文件到工作目录并改名为client_secret.json。  

##### 安装Google sheets API
```
pip install --upgrade google-api-python-client
```

##### set up the sample
```
from __future__ import print_function
import httplib2
import os

from apiclient import discovery
from oauth2client import client
from oauth2client import tools
from oauth2client.file import Storage

try:
    import argparse
    flags = argparse.ArgumentParser(parents=[tools.argparser]).parse_args()
except ImportError:
    flags = None

# If modifying these scopes, delete your previously saved credentials
# at ~/.credentials/sheets.googleapis.com-python-quickstart.json
SCOPES = 'https://www.googleapis.com/auth/spreadsheets.readonly'
CLIENT_SECRET_FILE = 'client_secret.json'
APPLICATION_NAME = 'Google Sheets API Python Quickstart'


def get_credentials():
    """Gets valid user credentials from storage.

    If nothing has been stored, or if the stored credentials are invalid,
    the OAuth2 flow is completed to obtain the new credentials.

    Returns:
        Credentials, the obtained credential.
    """
    home_dir = os.path.expanduser('~')
    credential_dir = os.path.join(home_dir, '.credentials')
    if not os.path.exists(credential_dir):
        os.makedirs(credential_dir)
    credential_path = os.path.join(credential_dir,
                                   'sheets.googleapis.com-python-quickstart.json')

    store = Storage(credential_path)
    credentials = store.get()
    if not credentials or credentials.invalid:
        flow = client.flow_from_clientsecrets(CLIENT_SECRET_FILE, SCOPES)
        flow.user_agent = APPLICATION_NAME
        if flags:
            credentials = tools.run_flow(flow, store, flags)
        else: # Needed only for compatibility with Python 2.6
            credentials = tools.run(flow, store)
        print('Storing credentials to ' + credential_path)
    return credentials

def main():
    """Shows basic usage of the Sheets API.

    Creates a Sheets API service object and prints the names and majors of
    students in a sample spreadsheet:
    https://docs.google.com/spreadsheets/d/1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms/edit
    """
    credentials = get_credentials()
    http = credentials.authorize(httplib2.Http())
    discoveryUrl = ('https://sheets.googleapis.com/$discovery/rest?'
                    'version=v4')
    service = discovery.build('sheets', 'v4', http=http,
                              discoveryServiceUrl=discoveryUrl)

    spreadsheetId = '1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms'
    rangeName = 'Class Data!A2:E'
    result = service.spreadsheets().values().get(
        spreadsheetId=spreadsheetId, range=rangeName).execute()
    values = result.get('values', [])

    if not values:
        print('No data found.')
    else:
        print('Name, Major:')
        for row in values:
            # Print columns A and E, which correspond to indices 0 and 4.
            print('%s, %s' % (row[0], row[4]))


if __name__ == '__main__':
    main()
```

##### run the sample
&ensp;&ensp;这个脚本执行时会打开浏览器获取verification code，而centos上没有安装浏览器，所以首次执行时需要加--noauth_local_webserver参数。执行后会输出一个链接并提示输入verification code，复制链接使用浏览器打开，复制里面的代码粘贴到程序即可。  

##### 参考资料
[Python Quickstart](https://developers.google.com/sheets/api/quickstart/python)