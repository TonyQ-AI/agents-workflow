---
name: workflow-installer
description: "涓€閿畨瑁?Workflow Task 宸ヤ綔娴佲€斺€旇銆岃宸ヤ綔娴併€嶈嚜鍔ㄦ悶瀹?
---

# Workflow 涓€閿畨瑁呭櫒

褰撶敤鎴疯銆岃宸ヤ綔娴併€嶃€乣瀹夎 Workflow Task`銆乣/workflow-installer` 鏃讹紝鎸変互涓嬫楠ゆ墽琛屻€?
**濡傛灉鐢ㄦ埛璇淬€屽崌绾у伐浣滄祦銆?*锛岃烦杞埌鏈€鍚庣殑銆屼竴閿崌绾с€嶉儴鍒嗐€?
## 涓€鍙ヨ瘽瀹夎娴佺▼

### 姝ラ1锛氬厠闅嗕粨搴?
```powershell
# 鍏嬮殕鍒颁复鏃剁洰褰?git clone --depth 1 https://github.com/TonyQ-AI/workflow-task.git $env:TEMP/workflow-task

# 濡傛灉 git clone 澶辫触锛堢綉缁滈棶棰橈級锛屽皾璇曚笅杞?ZIP锛?# Invoke-WebRequest -Uri "https://github.com/TonyQ-AI/workflow-task/archive/refs/heads/main.zip" -OutFile $env:TEMP/workflow.zip
# Expand-Archive $env:TEMP/workflow.zip $env:TEMP/workflow-installer
```

### 姝ラ2锛氬鍒舵妧鑳?
```powershell
# 澶嶅埗鎵€鏈?40 涓妧鑳藉埌椤圭洰
New-Item -Path ".reasonix/skills" -ItemType Directory -Force | Out-Null
Copy-Item -Recurse "$env:TEMP/workflow-installer/skills/*" ".reasonix/skills/" -Force
```

### 姝ラ3锛氶厤缃?AGENTS.md锛堣繘搴︽鏌ヨ鍒欙級

```powershell
$agentsRule = @"

## 杩涘害妫€鏌ヨ鍒?鍦ㄤ换浣曟搷浣滃紑濮嬩箣鍓嶏紝妫€鏌?docs/superpowers/plans/task_plan.md 鏄惁瀛樺湪銆?濡傛灉瀛樺湪锛岃鍙栧畠浠ヤ簡瑙ｅ綋鍓嶆墽琛岃繘搴︺€?杩欎竴瑙勫垯閫傜敤浜庢墍鏈夊璇濓紝涓嶈鏄惁浣跨敤 wf-orchestrator 缂栨帓鍣ㄣ€?"@
if (Test-Path "AGENTS.md") {
    Add-Content -Path "AGENTS.md" -Value $agentsRule
} else {
    Set-Content -Path "AGENTS.md" -Value $agentsRule
}
```

### 姝ラ4锛氶厤缃?MCP 鏈嶅姟鍣紙MiMo 澶氭ā鎬侊級

鍦?`reasonix.toml` 鎴栧搴旈厤缃枃浠朵腑娣诲姞锛?
```toml
[[plugins]]
name    = "mimo-multimodal"
command = "npx"
args    = ["-y", "tonyq-mimo-mcp-server"]
env     = {
  MIMO_API_URL  = "https://api.xiaomimimo.com/v1/chat/completions",
  MIMO_API_KEY = "${MIMO_API_KEY}"
}
call_timeout_seconds = 600
```

### 姝ラ5锛氶厤缃?API key

**鎻愮ず鐢ㄦ埛**璁剧疆浠ヤ笅鐜鍙橀噺锛堝瓨鍏?`~/.reasonix/.env`锛夛細

```env
DEEPSEEK_API_KEY=sk-xxxxx    # 宸ヤ綔娴佹帹鐞嗙敤锛堝繀濉級
MIMO_API_KEY=sk-xxxxx         # 澶氭ā鎬佸垎鏋愮敤锛堝彲閫夛級
```

濡傛灉鐢ㄦ埛鎻愪緵浜?key锛岀珛鍗冲啓鍏?`.env` 鏂囦欢锛堝厛瀛樺悗璇昏鍒欙級銆?
### 姝ラ6锛氶獙璇佸畨瑁?
纭浠ヤ笅鍏抽敭鏂囦欢宸插氨浣嶏細

```
- [ ] .reasonix/skills/wf-orchestrator/SKILL.md
- [ ] .reasonix/skills/wf-orchestrator-engine/SKILL.md
- [ ] .reasonix/skills/wf-architect/SKILL.md
- [ ] .reasonix/skills/wf-developer/SKILL.md
- [ ] .reasonix/skills/wf-tester/SKILL.md
- [ ] .reasonix/skills/wf-reviewer/SKILL.md
- [ ] .reasonix/skills/wf-deployer/SKILL.md
- [ ] AGENTS.md 鍖呭惈杩涘害妫€鏌ヨ鍒?```

### 姝ラ7锛氭姤鍛婄粨鏋?
```
鉁?Workflow Task 宸ヤ綔娴佸畨瑁呭畬鎴愶紒

宸插畨瑁?40 涓妧鑳斤細
  馃搵 鏍稿績宸ヤ綔娴? 鈥?8 涓紙缂栨帓鍣?寮曟搸+6瀛怉gent锛?  馃彌锔?鏋舵瀯璇勫/鎵弿 鈥?1 涓妧鑳斤紝2 绉嶇敤娉?  馃挕 鏂规硶璁烘敞鍏?鈥?9 涓紙TDD銆佺郴缁熷寲璋冭瘯绛夛級
  馃搻 瑙勫垝涓庢敮鎾?鈥?6 涓紙棰嗗煙寤烘ā銆佽繘搴﹁窡韪瓑锛?  馃敡 鎵╁睍鑳藉姏   鈥?6 涓紙UI璁捐銆佸伐浣滄祦鎵ц鍣ㄧ瓑锛?  馃帳 澶氭ā鎬?MCP 鈥?4 涓?MiMo 鎶€鑳斤紙鍚?tonyq-mimo-mcp-server锛?
馃搫 AGENTS.md 杩涘害妫€鏌ヨ鍒欏凡灏变綅
馃攲 MiMo MCP 宸查厤缃紙闇€璁剧疆 MIMO_API_KEY 鐜鍙橀噺锛?
浣跨敤鏂瑰紡锛?  /wf-orchestrator <椤圭洰鍚?: <闇€姹傛弿杩?

璇︾粏鏂囨。锛?  https://github.com/TonyQ-AI/workflow-task
```

## 涓€閿崌绾э紙閽堝宸插畨瑁呯敤鎴凤級

褰撶敤鎴疯銆屽崌绾у伐浣滄祦銆嶆椂锛岃嚜鍔ㄦ墽琛屼互涓嬫搷浣滐細

### 1. 鎷夊彇鏈€鏂版妧鑳?```powershell
git clone --depth 1 https://github.com/TonyQ-AI/workflow-task.git $env:TEMP/rw-upgrade
Copy-Item -Recurse "$env:TEMP/rw-upgrade/skills/*" ".reasonix/skills/" -Force
```

### 2. 鏇存柊 MCP 閰嶇疆锛堝叧閿慨澶嶏級
```powershell
$configPath = "reasonix.toml"
if (Test-Path $configPath) {
    $content = Get-Content $configPath -Raw
    $content = $content -replace 'mimo-mcp-server', 'tonyq-mimo-mcp-server'
    $content = $content -replace 'MIMO_API_BASE', 'MIMO_API_URL'
    $content = $content -replace 'MIMO_API_URL = "https://api.xiaomimimo.com/v1"', 'MIMO_API_URL = "https://api.xiaomimimo.com/v1/chat/completions"'
    Set-Content -Path $configPath -Value $content -NoNewline
    Write-Output "MCP 閰嶇疆宸插崌绾?
}
```

### 3. 娓呴櫎 MCP 缂撳瓨
```powershell
$cacheFile = "$env:LOCALAPPDATA\reasonix\mcp\mimo-multimodal.json"
if (Test-Path $cacheFile) { Remove-Item $cacheFile -Force; Write-Output "MCP 缂撳瓨宸叉竻闄? }
```

### 4. 鏇存柊 AGENTS.md
```powershell
$agentsCheck = Select-String -Path "AGENTS.md" -Pattern "杩涘害妫€鏌ヨ鍒? -SimpleMatch -ErrorAction SilentlyContinue
if (-not $agentsCheck) { Add-Content -Path "AGENTS.md" -Value "`n## 杩涘害妫€鏌ヨ鍒檂n鍦ㄤ换浣曟搷浣滃紑濮嬩箣鍓嶏紝妫€鏌?docs/superpowers/plans/task_plan.md 鏄惁瀛樺湪銆俙n濡傛灉瀛樺湪锛岃鍙栧畠浠ヤ簡瑙ｅ綋鍓嶆墽琛岃繘搴︺€? }
```

### 5. 鎶ュ憡鍗囩骇缁撴灉
```
宸ヤ綔娴佸崌绾у畬鎴愶紒
  鎶€鑳藉凡鏇存柊鍒版渶鏂扮増锛?0 涓級
  MCP 閰嶇疆锛歮imo-mcp-server 鈫?tonyq-mimo-mcp-server
  鐜鍙橀噺锛歁IMO_API_BASE 鈫?MIMO_API_URL锛堝甫 /chat/completions锛?  MCP 缂撳瓨宸叉竻闄?
璇烽噸鍚?Reasonix 浣挎柊閰嶇疆鐢熸晥
```

## 娉ㄦ剰浜嬮」

- 瀹夎鍓嶄細妫€娴嬫槸鍚﹀凡瀛樺湪鎶€鑳斤紝涓嶈鐩栫敤鎴疯嚜瀹氫箟淇敼
- AGENTS.md 浠ヨ拷鍔犳柟寮忓啓鍏ワ紝涓嶈鐩栧凡鏈夊唴瀹?- 濡傛灉 git clone 澶辫触浼氳嚜鍔ㄥ垏鍒?ZIP 涓嬭浇妯″紡
- 瀹夎瀹屾垚鍚?*寤鸿閲嶅惎 Reasonix** 浣挎柊鎶€鑳界敓鏁?- MiMo MCP 闇€瑕?`MIMO_API_KEY` 鐜鍙橀噺锛堝瓨 `.env` 涓嶇‖缂栫爜锛?