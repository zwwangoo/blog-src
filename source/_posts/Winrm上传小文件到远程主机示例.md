---
title: Winrm上传小文件到远程主机示例
date: 2025-12-23
tags:
  - 代码片段
---

```python
import winrm
import binascii
import os
import hashlib

def calculate_local_md5(file_path):
    """计算本地文件的 MD5"""
    hash_md5 = hashlib.md5()
    with open(file_path, "rb") as f:
        for chunk in iter(lambda: f.read(4096), b""):
            hash_md5.update(chunk)
    return hash_md5.hexdigest().upper()

def upload_file_fixed(host, username, password, local_path, remote_path):
    # 建立 WinRM 会话
    session = winrm.Session(host, auth=(username, password), transport='ntlm')

    file_name = os.path.basename(local_path)
    
    # 修复语法错误：不直接在 f-string 大括号内写反斜杠
    r_path = remote_path.rstrip('\\')
    remote_full_path = r_path + "\\" + file_name
    tmp_path = remote_full_path + ".hex"

    # 1. 计算本地 MD5
    print(f"[*] 正在计算本地文件 MD5...")
    local_md5 = calculate_local_md5(local_path)
    print(f"    本地 MD5: {local_md5}")

    print(f"[*] 准备上传: {local_path} -> {remote_full_path}")
    
    # 2. 读取本地文件并转为 Hex
    with open(local_path, "rb") as f:
        hex_content = binascii.hexlify(f.read()).decode('utf-8')

    # 3. 清理远程旧文件
    session.run_ps(f'if (Test-Path "{tmp_path}") {{ Remove-Item "{tmp_path}" -Force }}')
    session.run_ps(f'if (Test-Path "{remote_full_path}") {{ Remove-Item "{remote_full_path}" -Force }}')
    
    # 4. 分块上传 Hex
    # 进一步缩小到 3000，解决命令过长问题
    chunk_size = 3000  
    total_len = len(hex_content)
    print(f"[*] Hex长度: {total_len}，开始分块追加...")

    for i in range(0, total_len, chunk_size):
        chunk = hex_content[i:i+chunk_size]
        # 使用 ac (Add-Content) 极简命令
        cmd = 'ac "' + tmp_path + '" "' + chunk + '" -NoNewline'
        r_chunk = session.run_ps(cmd)
        
        if r_chunk.status_code != 0:
            print(f"\n [!] 分块追加失败 (偏移: {i}):")
            print(f" 错误信息: {r_chunk.std_err.decode('utf-8')}")
            return
        
        if (i // chunk_size) % 10 == 0:
            print(f"[+] 进度: {min(i + chunk_size, total_len)} / {total_len}")

    # 5. 还原并校验
    print("[*] 正在远程还原文件并进行校验...")
    decode_script = f"""
    try {{
        $hex = gc "{tmp_path}" -Raw
        $hex = $hex.Trim()
        $bytes = New-Object byte[] ($hex.Length / 2)
        for($i=0; $i -lt $hex.Length; $i+=2){{
            $bytes[$i/2] = [Convert]::ToByte($hex.Substring($i, 2), 16)
        }}
        [System.IO.File]::WriteAllBytes("{remote_full_path}", $bytes)
        ri "{tmp_path}" -Force
        (Get-FileHash "{remote_full_path}" -Algorithm MD5).Hash
    }} catch {{
        $_.Exception.Message
    }}
    """
    
    r = session.run_ps(decode_script)
    
    if r.status_code == 0:
        remote_md5 = r.std_out.decode('utf-8').strip()
        # 兼容性处理：如果返回的是对象格式，提取最后一行
        if "\n" in remote_md5:
            remote_md5 = remote_md5.splitlines()[-1].strip()

        print(f"    远程 MD5: {remote_md5}")
        
        if local_md5.upper() == remote_md5.upper():
            print(f"\n [OK] 校验通过！文件一致。")
        else:
            print(f"\n [!] 校验失败！MD5 不匹配。")
            print(f"      本地: {local_md5}")
            print(f"      远程: {remote_md5}")
    else:
        print(f" [!] 远程还原失败: {r.std_err.decode('utf-8')}")

# --- 配置参数 ---
target_host = '192.168.xxx.xxx'
user = 'administrator'
pwd = 'xxxxxx'
local_file = 'test.exe'
remote_dir = 'C:\\'

if __name__ == "__main__":
    if os.path.exists(local_file):
        upload_file_fixed(target_host, user, pwd, local_file, remote_dir)
    else:
        print(f"错误: 找不到本地文件 {local_file}")
```