# Multi-Agent Automated Workflow

> 澶欰gent鍗忓悓寮€鍙戝叏娴佺▼鑷姩鍖栧伐浣滄祦 路 鍙屾ā寮?路 40 涓妧鑳?路 13 闃舵

## 鍙屾ā寮忚嚜鍔ㄥ垏鎹?
鏈伐浣滄祦鑷姩妫€娴嬪钩鍙拌兘鍔涳紝鏃犻渶鎵嬪姩閫夋嫨锛?
| 妯″紡 | 鏉′欢 | 鎵ц鏂瑰紡 |
|------|------|---------|
| **姝ｅ父妯″紡** | task 宸ュ叿鍙敤 | 缂栨帓鍣?鈫?task(寮曟搸 subagent) 鈫?task(瀛怉gent) |
| **inline 妯″紡** | task 涓嶅彲鐢?| 缂栨帓鍣ㄧ洿鎺ユ墽琛屾墍鏈夐樁娈?|

鍚姩鏃惰嚜鍔ㄨ緭鍑哄綋鍓嶆ā寮忥細
```
鉁?姝ｅ父妯″紡 鈥?閫氳繃 task 娲惧彂瀛怉gent
鈿狅笍 inline 妯″紡 鈥?task 涓嶅彲鐢紝缂栨帓鍣ㄧ洿鎺ユ墽琛?```

## 涓€鍙ヨ瘽瀹夎

```
瑁呭伐浣滄祦
```

## 浣跨敤

```
/wf-orchestrator 椤圭洰鍚? 闇€姹傛弿杩?/wf-orchestrator 椤圭洰鍚? 闇€姹傛弿杩?--lite
/wf-orchestrator 椤圭洰鍚? 闇€姹傛弿杩?--from coding
```

## 椤圭洰鍏崇郴

```
workflow-installer锛圧easonix 涓撶敤 路 40鎶€鑳斤級
       鈫?鏀归€?workflow-inline锛堢函 inline 路 39鎶€鑳?路 鏃?task 渚濊禆锛?       鈫?鍚堝苟
Multi-Agent Automated Workflow锛堝弻妯″紡 路 40鎶€鑳?路 鑷€傚簲锛?```

## 閰嶇疆

- `MIMO_API_URL`锛歚https://api.xiaomimimo.com/v1/chat/completions`
- `MIMO_API_KEY`锛氬瓨鍏?`.env`锛岄厤缃紩鐢?`${MI*******KEY}`
- `DEEPSEEK_API_KEY`锛氬伐浣滄祦鎺ㄧ悊
