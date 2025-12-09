# Music Scraper 激活码生成指南

## 一、概述

本系统采用**离线激活码**机制，无需服务器验证。激活码与设备码绑定，确保一码一机。

### 授权模式

| 模式 | 试用期 | 激活后 | 过期后 |
|------|--------|--------|--------|
| 手动刮削 | ✅ 可用 | ✅ 可用 | ✅ 可用 |
| 自动刮削 | ✅ 可用 | ✅ 可用 | ❌ 受限 |
| 批量刮削 | ✅ 可用 | ✅ 可用 | ❌ 受限 |

- **试用期**：首次安装后 30 天
- **激活码有效期**：默认 365 天，可自定义
- **永久激活码**：无过期时间

## 二、获取用户设备码

用户在 **设置页面** 可以看到自己的设备码，格式如：`DV3AE91EB0`

用户需将设备码发送给管理员以生成激活码。

## 三、生成激活码

### 3.1 使用命令行工具

```bash
cd /path/to/music4docker

# 生成 365 天有效期激活码（默认）
python3 tools/generate_license.py --device DV3AE91EB0

# 生成指定天数的激活码
python3 tools/generate_license.py --device DV3AE91EB0 --days 90

# 生成永久激活码
python3 tools/generate_license.py --device DV3AE91EB0 --permanent

# 验证激活码
python3 tools/generate_license.py --verify 3AE9-1EB0-CB01-A565
```

### 3.2 交互式生成

```bash
python3 tools/generate_license.py
```

按提示输入设备码和有效期。

### 3.3 使用 Python 代码生成

```python
from backend.license import LicenseManager

lm = LicenseManager()

# 生成 365 天激活码
result = lm.generate_license_key('DV3AE91EB0', valid_days=365)
print('激活码:', result['license_key'])
print('过期时间:', result['expires_at'])

# 生成永久激活码
result = lm.generate_license_key('DV3AE91EB0', permanent=True)
print('永久激活码:', result['license_key'])
```

## 四、激活码格式

```
XXXX-XXXX-XXXX-XXXX
```

示例：`3AE9-1EB0-CB01-A565`

| 部分 | 说明 |
|------|------|
| 第1-2段 | 设备码编码 |
| 第3段 | 日期/类型编码（PERM 表示永久） |
| 第4段 | 校验码 |

## 五、用户激活流程

1. 用户打开 **设置页面**
2. 复制 **设备码** 发送给管理员
3. 管理员生成激活码并发送给用户
4. 用户在设置页面输入激活码，点击 **激活**
5. 激活成功后显示剩余天数或"永久"

## 六、常见问题

### Q1: 激活码提示"与当前设备不匹配"

激活码是根据设备码生成的，只能在对应设备上使用。请确认：
- 用户提供的设备码是否正确
- 是否在正确的设备上激活

### Q2: 如何延长有效期？

为用户生成新的激活码，用户输入新激活码即可覆盖旧的。

### Q3: 用户重装后设备码变了怎么办？

设备码存储在 `/app/data/device.json`，如果数据目录被清空，会生成新设备码。

**解决方案**：
- 提醒用户备份 `data` 目录
- 或为新设备码生成新激活码

### Q4: 如何批量生成激活码？

```python
from backend.license import LicenseManager

lm = LicenseManager()
device_codes = ['DV12345678', 'DV87654321', 'DVABCDEFGH']

for device in device_codes:
    result = lm.generate_license_key(device, valid_days=365)
    print(f'{device}: {result["license_key"]}')
```

## 七、安全说明

1. **密钥保护**：`SECRET_KEY` 位于 `backend/license.py`，请勿泄露
2. **激活码不可逆**：无法从激活码反推原始信息
3. **设备绑定**：激活码仅在对应设备有效
4. **本地验证**：所有验证在本地完成，无网络请求

## 八、配置参数

位于 `backend/license.py`：

```python
class LicenseManager:
    SECRET_KEY = "MS2025_CyberPunk_NeonGlow_X7K9"  # 加密密钥
    TRIAL_DAYS = 30           # 试用期天数
    LICENSE_VALID_DAYS = 365  # 默认激活码有效天数
```

如需修改默认参数，请在打包前调整。

