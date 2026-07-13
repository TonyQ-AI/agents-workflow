# Workflow Task 瀹屾暣鎶€鏈墜鍐?
> 鐗堟湰锛?.1.2 路 鏈€鍚庢洿鏂帮細2026-07-13

---

## 鐩綍

1. [鏋舵瀯姒傝](#1-鏋舵瀯姒傝)
2. [鍙屽紩鎿庤璁(#2-鍙屽紩鎿庤璁?
3. [瀹屾暣鎵ц娴佺▼](#3-瀹屾暣鎵ц娴佺▼)
4. [纭棬鎺ф満鍒禲(#4-纭棬鎺ф満鍒?
5. [鍏滃簳涓庨檷绾(#5-鍏滃簳涓庨檷绾?
6. [杩涘害璺熻釜涓庢柇鐐圭画璺慮(#6-杩涘害璺熻釜涓庢柇鐐圭画璺?
7. [鍘熷闇€姹備氦鍙夐獙璇乚(#7-鍘熷闇€姹備氦鍙夐獙璇?
8. [鐭ヨ瘑娌夋穩](#8-鐭ヨ瘑娌夋穩)
9. [瀛怉gent璋冨害](#9-瀛恆gent璋冨害)
10. [MCP 澶氭ā鎬侀泦鎴怾(#10-mcp-澶氭ā鎬侀泦鎴?
11. [鎶€鑳界洰褰昡(#11-鎶€鑳界洰褰?
12. [v4.x 鐗堟湰婕旇繘](#12-v4x-鐗堟湰婕旇繘)

---

## 1. 鏋舵瀯姒傝

### 1.1 涓€鍙ヨ瘽鎻忚堪

鐢ㄦ埛杈撳叆涓€鍙ヨ瘽闇€姹傦紝缂栨帓鍣ㄨ嚜鍔ㄤ覆璧蜂粠闇€姹傛緞娓呭埌鐗堟湰鐧昏鐨勫叏娴佺▼銆?
### 1.2 鏁翠綋鏋舵瀯

```
鐢ㄦ埛: /wf-orchestrator MediaManager: 鏂板鎵归噺瀵煎嚭

缂栨帓鍣?(wf-orchestrator) 鈥?涓讳細璇?  鈹溾攢鈹€ 瑙ｆ瀽鍙傛暟 + 鍒涘缓浼氳瘽鐩綍
  鈹溾攢鈹€ 浜や簰寮忛渶姹傛緞娓咃紙涓庣敤鎴蜂竴闂竴绛旓級
  鈹溾攢鈹€ 浜у嚭: task_plan.md + specs/*.md
  鈹斺攢鈹€ 鎵撳寘 JSON -> task(wf-orchestrator-engine)

鎵ц寮曟搸 (wf-orchestrator-engine) 鈥?鐙珛涓婁笅鏂?  鈹溾攢鈹€ 鎺ユ敹 JSON 鍖呰９
  鈹溾攢鈹€ 鍚姩鑷锛堟樉绀洪」鐩?妯″紡/鑼冨洿锛?  鈹溾攢鈹€ 12椤逛换鍔′緷娆℃墽琛?  鈹斺攢鈹€ 杩斿洖 JSON 鎽樿缁欑紪鎺掑櫒
```

### 1.3 v4.x 鏍稿績璁捐鍐崇瓥

| 璁捐鍐崇瓥 | 鍘熷洜 |
|:---------|:-----|
| 鍙屽紩鎿庢媶鍒?| 缂栨帓鍣ㄤ氦浜掍骇鐢熷ぇ閲忎笂涓嬫枃锛屽紩鎿庣嫭绔嬩細璇濅笉鍙楀共鎵?|
| 浠诲姟娓呭崟鏇夸唬闃舵缂栧彿 | 娑堥櫎璁℃暟鐭涚浘銆侫I涓嶆暟"绗嚑涓?锛屽彧鍋?涓嬩竴椤? |
| 纭棬鎺?+ 鍥為€€涓婇檺3娆?| 鏋舵瀯闂蹇呴』淇紝浣嗕笉鑳芥棤闄愬惊鐜?|
| fallback鍏滃簳 | task()涓嶅彲鐢ㄦ椂寮曟搸鐩存墽琛岋紝宸ヤ綔娴佷笉涓柇 |
| 鍘熸枃浜ゅ弶楠岃瘉 | 姣忓眰瀛怉gent瀵圭収鐢ㄦ埛纭鐨剆pecs鏍￠獙锛岄槻闇€姹傛紓绉?|

---

## 2. 鍙屽紩鎿庤璁?
### 2.1 涓轰粈涔堟媶鍒嗭紙v3.x -> v4.0锛?
v3.x 鍗曞紩鎿庡瓨鍦ㄤ笁涓棶棰橈細

1. **涓婁笅鏂囪啫鑳€** 鈥?瑙勫垝闃舵50杞棶绛斿悗锛屾墽琛岄樁娈佃繕瑕佽儗杩欐鍘嗗彶
2. **瀛怉gent娣蜂贡** 鈥?瀛怉gent鍙兘缈诲埌鍓嶉潰瀵硅瘽锛岃瑙ｅ綋鍓嶄换鍔?3. **鎭㈠鍥伴毦** 鈥?涓柇鍚庤浠庡ご鍥炴函鏁翠釜瀵硅瘽鍘嗗彶

### 2.2 寮曟搸1锛氱紪鎺掑櫒 (wf-orchestrator) 

鍥涙鎿嶄綔锛?
- **姝ラ1** 鈥?鍙傛暟瑙ｆ瀽銆佸垱寤轰細璇濈洰褰曘€佹娴嬩腑鏂仮澶?- **姝ラ2** 鈥?浜や簰寮忛渶姹傛緞娓咃紙brainstorming閫愯疆闂瓟锛夛紝浜у嚭 task_plan.md + specs
- **姝ラ3** 鈥?鎵撳寘JSON -> task(wf-orchestrator-engine)
- **姝ラ4** 鈥?鎺ユ敹寮曟搸杩斿洖锛屽睍绀虹粨鏋?
### 2.3 寮曟搸2锛氭墽琛屽紩鎿?(wf-orchestrator-engine)

12椤逛换鍔℃竻鍗曪細

1. 棰嗗煙寤烘ā  2. 鏋舵瀯璁捐  3. 鏋舵瀯璇勫(纭棬鎺?  4. 缂栫爜
5. 娴嬭瘯  6. 鏋舵瀯鎵弿(纭棬鎺?  7. 瀹℃煡  8. 閮ㄧ讲
9. 鏂囨。妫€鏌? 10. 鍒嗘敮鏀跺熬  11. 鐗堟湰鐧昏  12. 鎬荤粨+鐭ヨ瘑娌夋穩

鍚姩鏃惰嚜妫€杈撳嚭鍏抽敭淇℃伅锛堥」鐩?妯″紡/鑼冨洿锛夛紝fallback鏃舵樉寮忓憡鐭ョ敤鎴枫€?
### 2.4 v4.0 -> v4.1 涓昏婕旇繘

| | v4.0 | v4.1 |
|:--|:-----|:-----|
| 鎻忚堪鏍煎紡 | "闃舵 4/13锛氭灦鏋勮瘎瀹? | ">>> 鏋舵瀯璇勫" |
| 璁℃暟鏂瑰紡 | 缂栧彿锛堝父鍑洪敊锛?| 浠诲姟娓呭崟锛堜笉鏁版暟锛?|
| 璇勫垎鏍囧噯 | 35/60 | 18/24锛堢粺涓€锛?|
| --to | 澹版槑鏈疄鐜?| 姣忎釜浠诲姟妫€鏌?|
| --parallel | 澹版槑鏈疄鐜?| 浠诲姟4/5骞惰璋冨害 |
| --timeout | 纭紪鐮?00 | 鍙傛暟鍖?|
| 鍥為€€涓婇檺 | 鏃?| 鏈€澶?娆?|
| fallback | 纭紪鐮乫alse | 澶辫触2娆¤嚜鍔ㄩ檷绾?|

---

## 3. 瀹屾暣鎵ц娴佺▼

### 3.1 姝ｅ父妯″紡锛堜娇鐢ㄦ敮鎸?task 鐨勫伐鍏锋椂锛?
缂栨帓鍣ㄥ畬鎴愪氦浜掑紡瑙勫垝鍚庯紝灏?JSON 鍖呰９浼犵粰鎵ц寮曟搸锛屽紩鎿庡湪鐙珛涓婁笅鏂囦腑鎸夋竻鍗曟墽琛岋細

```
鍚姩鑷 -> 鏄剧ず椤圭洰/妯″紡/鑼冨洿 -> 閫愰」鎵ц
```

**浠诲姟1: 棰嗗煙寤烘ā** 鈥?寮曟搸鐩存墽琛?- 璇诲彇 specs + task_plan
- 娉ㄥ叆 domain-modeling 鏂规硶璁?- 鎻愬彇瀹炰綋銆佸畾涔夊叧绯汇€佸缓绔嬮€氱敤璇█
- 浜у嚭: domain-model.md
- 浜ゅ弶楠岃瘉: 姒傚康鏄惁涓巗pecs涓€鑷?
**浠诲姟2: 鏋舵瀯璁捐** 鈥?task(wf-architect) Pro妯″瀷
- 浼犲叆 specs + domain-model + task_plan + 椤圭洰鏍圭洰褰?- 濡備负鍥為€€閲嶈瘯锛屾敞鍏ヤ笂娆¤瘎瀹℃剰瑙?- 浜у嚭: 02-design.md

**浠诲姟3: 鏋舵瀯璇勫(纭棬鎺?** 鈥?寮曟搸鐩存墽琛?- 娉ㄥ叆 arch-review锛?缁磋瘎鍒?婊″垎24)
- Go(>=18): 浜у嚭 03-arch-review.md锛岃繘浠诲姟4
- No-Go(<18): 鍥為€€浠诲姟2锛屾渶澶?娆★紝瓒呴檺寮哄埗Go

**浠诲姟4: 缂栫爜** 鈥?task(wf-developer) Flash妯″瀷
- 鍚姩鍓嶉獙璇? 03-arch-review.md 蹇呴』瀛樺湪涓斾负Go
- 浼犲叆 specs + design + task_plan + 椤圭洰鏍圭洰褰?- --parallel鏃跺悓鏃跺惎鍔ㄤ换鍔?
- 娉ㄥ叆 TDD/绯荤粺鍖栬皟璇?UI璁捐(鎸夐渶)
- 浜у嚭: 浠ｇ爜 + CHANGES.md

**浠诲姟5: 娴嬭瘯** 鈥?task(wf-tester) Flash妯″瀷
- --parallel鏃剁瓑寰呭凡鍦ㄤ换鍔?鍚姩鐨勬祴璇曞畬鎴?- 娉ㄥ叆 dispatching-parallel-agents + verification
- 浜у嚭: TEST_REPORT.md

**浠诲姟6: 鏋舵瀯鎵弿(纭棬鎺?** 鈥?寮曟搸鐩存墽琛?- 鎵弿浠ｇ爜缁撴瀯锛屾娴嬪惊鐜緷璧栥€佹ā鍧楄€﹀悎
- 鏃犱弗閲嶉棶棰? 浜у嚭 architecture-scan.html锛岃繘浠诲姟7
- 鏈変弗閲嶉棶棰? 鍥為€€浠诲姟4锛屾渶澶?娆?
**浠诲姟7: 瀹℃煡** 鈥?task(wf-reviewer) Pro妯″瀷
- 鍚姩鍓嶉獙璇? architecture-scan.html 蹇呴』瀛樺湪
- 娉ㄥ叆浠ｇ爜瀹℃煡 + 涓枃瀹℃煡 + 鍙嶉澶勭悊
- 浜у嚭: 05-review.md

**浠诲姟8: 閮ㄧ讲** 鈥?task(wf-deployer) Flash妯″瀷
- 妫€娴嬮」鐩被鍨嬶紝鎵ц鏋勫缓鍛戒护
- 浜у嚭: DEPLOY.md

**浠诲姟9: 鏂囨。妫€鏌?* 鈥?寮曟搸鐩存墽琛?- 娉ㄥ叆 chinese-documentation
- 妫€鏌ユ帓鐗堛€佹湳璇€佹爣鐐?- OCR澶勭悊鍥剧墖鍨嬫枃妗?- 浜у嚭: doc-check-report.md

**浠诲姟10: 鍒嗘敮鏀跺熬** 鈥?寮曟搸鐩存墽琛?- git add + chinese-commit-conventions 鏍煎紡鎻愪氦
- 璇㈤棶鏄惁鎺ㄩ€?鍒涘缓PR
- 娉ㄥ叆 finishing-a-development-branch + git-worktrees

**浠诲姟11: 鐗堟湰鐧昏** 鈥?寮曟搸鐩存墽琛?- 娉ㄥ叆 vt锛屾洿鏂?.vt.json

**浠诲姟12: 鐢熸垚鎬荤粨** 鈥?寮曟搸鐩存墽琛?- 璇诲彇鎵€鏈変骇鍑烘憳瑕侊紝鍐欏叆 SUMMARY.md
- 鐭ヨ瘑娌夋穩: 鎻愮偧鍏抽敭鍙戠幇鍐欏叆 KNOWLEDGE.md
- 杩斿洖 JSON 鎽樿缁欑紪鎺掑櫒

### 3.2 纭棬鎺т粙鍏?
浠诲姟3锛堟灦鏋勮瘎瀹★級No-Go -> 鍥為€€浠诲姟2锛堟渶澶?娆★級
浠诲姟6锛堟灦鏋勬壂鎻忥級涓ラ噸闂 -> 鍥為€€浠诲姟4锛堟渶澶?娆★級
浠诲姟4鍚姩鍓嶆鏌?3-arch-review.md蹇呴』Go
浠诲姟7鍚姩鍓嶆鏌rchitecture-scan.html蹇呴』瀛樺湪

### 3.3 鍏滃簳妯″紡

fallback=true -> 鎵€鏈塼ask()鏇挎崲涓哄紩鎿庣洿鎵ц锛岃川閲忕暐闄嶄絾涓嶄腑鏂?
### 3.4 --lite 杞婚噺妯″紡

璺宠繃浠诲姟3锛堟灦鏋勮瘎瀹★級+ 浠诲姟6锛堟灦鏋勬壂鎻忥級锛岄€傚悎灏忓瀷淇

### 3.5 --parallel 骞惰

浠诲姟4/5鍚屾椂鍚姩 wf-developer + wf-tester

### 3.6 鍚勪换鍔′骇鍑?
| 浠诲姟 | 浜у嚭鏂囦欢 | 鎵ц鑰?|
|:-----|:--------|:------|
| 1 棰嗗煙寤烘ā | domain-model.md | 寮曟搸 |
| 2 鏋舵瀯璁捐 | 02-design.md | wf-architect |
| 3 鏋舵瀯璇勫 | 03-arch-review.md | 寮曟搸 |
| 4 缂栫爜 | 浠ｇ爜 + CHANGES.md | wf-developer |
| 5 娴嬭瘯 | TEST_REPORT.md | wf-tester |
| 6 鏋舵瀯鎵弿 | architecture-scan.html | 寮曟搸 |
| 7 瀹℃煡 | 05-review.md | wf-reviewer |
| 8 閮ㄧ讲 | DEPLOY.md | wf-deployer |
| 9 鏂囨。妫€鏌?| doc-check-report.md | 寮曟搸 |
| 10 鍒嗘敮鏀跺熬 | git commit + PR | 寮曟搸 |
| 11 鐗堟湰鐧昏 | .vt.json | 寮曟搸 |
| 12 鎬荤粨 | SUMMARY.md | 寮曟搸 |

---

## 4. 纭棬鎺ф満鍒?
### 4.1 鏋舵瀯璇勫闂ㄦ帶锛堜换鍔?锛?
6缁磋瘎鍒嗭紝姣忕淮1-4鍒嗭紝婊″垎24銆侴o: >=18/24 (75%)

| 缁村害 | 璇存槑 |
|:-----|:-----|
| 鍔熻兘瀹屾暣鎬?| 鏄惁瑕嗙洊鎵€鏈夐渶姹傜偣 |
| 鍙墿灞曟€?| 鏋舵瀯鏄惁鏀寔鏈潵鎵╁睍 |
| 鎬ц兘 | 鏄惁鏈夋€ц兘鑰冮噺 |
| 瀹夊叏鎬?| 瀹夊叏璁捐 |
| 鍙淮鎶ゆ€?| 娓呮櫚銆佸彲娴嬭瘯 |
| 鎴愭湰 | 鎶€鏈€夊瀷鍚堢悊 |

No-Go鏃跺洖閫€浠诲姟2锛堟渶澶?娆★紝瓒呴檺寮哄埗Go+璁板綍椋庨櫓锛?
### 4.2 鏋舵瀯鎵弿闂ㄦ帶锛堜换鍔?锛?
妫€娴嬪惊鐜緷璧栥€佹ā鍧楄€﹀悎銆佹灦鏋勮厫鍖栥€備弗閲嶉棶棰樺洖閫€浠诲姟4锛堟渶澶?娆★級

### 4.3 鍙岄噸楠岃瘉

- 浠诲姟4鍚姩鍓? 蹇呴』03-arch-review.md瀛樺湪涓斾负Go
- 浠诲姟7鍚姩鍓? 蹇呴』architecture-scan.html瀛樺湪
- 涓嶆弧瓒冲垯鍥為€€鍒板搴斾慨澶嶄换鍔?
---

## 5. 鍏滃簳涓庨檷绾?
### 瑙﹀彂鏉′欢
1. 缂栨帓鍣ㄦ娴媡ask()鍙敤鎬?2. task()杩炵画澶辫触2娆¤嚜鍔ㄩ檷绾?
### 鍚姩鍛婄煡
```
姝ｅ父: 鎵ц妯″紡: 姝ｅ父妯″紡锛坱ask娲惧彂瀛怉gent锛?鍏滃簳: 鎵ц妯″紡: 鍏滃簳妯″紡锛坱ask涓嶅彲鐢紝寮曟搸鐩存墽琛岋級
      褰撳墠鐜涓嶆敮鎸乼ask()瀛怉gent璋冪敤锛屽凡鍒囨崲涓哄厹搴曟ā寮?```

宸ュ叿: ZCode/Reasonix 鏈塼ask()锛屽叾浠栬嚜鍔╢allback

---

## 6. 杩涘害璺熻釜涓庢柇鐐圭画璺?
涓夋枃浠? task_plan.md + progress.md + findings.md
鍔?checkpoint.json 璁板綍鏂偣

--from 鎭㈠鏃堕獙璇佸墠缃枃浠跺瓨鍦?--to 鍛戒腑璺充换鍔?2
闃舵鍚嶆槧灏? planning > domain_modeling > architecture > arch_review > coding > testing > arch_scan > review > deploy > doc_check > branch_finish > version_tracking

progress鏍囩鏄犲皠琛ㄧ‘淇濅腑鏂囨爣绛句笌鑻辨枃checkpoint鍊煎搴?
---

## 7. 鍘熷闇€姹備氦鍙夐獙璇?
姣忎釜瀛怉gent鏀跺埌涓や唤杈撳叆: 鍓嶅簭琛嶇敓鏂囨。(鎵夸笂) + 鐢ㄦ埛纭鐨剆pecs(鏍￠獙)
鍋忕鏃舵樉寮忔爣娉ㄣ€傞槻闇€姹傚眰灞備紶閫掍腑婕傜Щ

---

## 8. 鐭ヨ瘑娌夋穩

姣忔浼氳瘽缁撴潫鎻愮偧 -> 鍐欏叆 KNOWLEDGE.md -> 涓嬫鍚姩鑷姩娉ㄥ叆
鏍煎紡: 鍏抽敭鍐崇瓥/缁忛獙鏁欒/棰嗗煙鐭ヨ瘑/娉ㄦ剰浜嬮」/鏁版嵁娲炲療

---

## 9. 瀛怉gent璋冨害

5涓猼ask瀛怉gent: wf-architect/wf-developer/wf-tester/wf-reviewer/wf-deployer
Pro妯″瀷(鏋舵瀯/瀹℃煡) Flash妯″瀷(缂栫爜/娴嬭瘯/閮ㄧ讲)
璋冪敤鏍煎紡: task(subagent_name, prompt, model, timeout)
浜у搧楠岃瘉: 1閲嶈瘯2鏍囪澶辫触3缁х画

---

## 10. MCP 澶氭ā鎬侀泦鎴?
### 10.1 褰撳墠閰嶇疆

```toml
[[plugins]]
name    = "mimo-multimodal"
command = "npx"
args    = ["-y", "tonyq-mimo-mcp-server"]
env     = {
  MIMO_API_URL  = "https://api.xiaomimimo.com/v1/chat/completions",
  MIMO_API_KEY  = "${MI*******KEY}"
}
call_timeout_seconds = 600
```

**MiMo 宸ュ叿娓呭崟锛?涓級锛?*

| MCP 宸ュ叿鍚?| 瀵瑰簲鎶€鑳?| 鐢ㄩ€?|
|:-----------|:---------|:-----|
| `mimo_understand_image` | mimo-image-understanding | 鍥剧墖鍒嗘瀽/OCR/UI瀹℃煡 |
| `mimo_understand_audio` | mimo-audio-understanding | 闊抽杞啓/鐞嗚В |
| `mimo_understand_video` | mimo-video-understanding | 瑙嗛鍐呭鍒嗘瀽 |
| `mimo_speech_recognition` | 鈥?| 璇煶璇嗗埆(ASR) |
| `mimo_speech_synthesis` | mimo-tts | 棰勭疆闊宠壊璇煶鍚堟垚 |
| `mimo_voice_design` | mimo-tts | 鏂囨湰鎻忚堪鑷畾涔夐煶鑹?|
| `mimo_voice_clone` | mimo-tts | 闊抽鏍锋湰鍏嬮殕闊宠壊 |

### 10.2 v4.1.2 杩佺Щ璇存槑

**涓轰粈涔堜粠 `mimo-mcp-server` 鎹㈡垚 `tonyq-mimo-mcp-server`锛?*

鍘?`mimo-mcp-server` 鐢辩涓夋柟 alionsss 鍙戝竷銆俷pm 鍙戝竷鐗堬紙v0.1.2锛夌殑宸ュ叿鍚嶅姞浜?`mimo_` 鍓嶇紑锛堝 `mimo_understand_image`锛夛紝浣?GitHub 婧愮爜鏈悓姝ワ紝浠嶇劧鐢ㄦ棫鍚嶏紙`understand_image`锛夈€傚鑷?npx 鎷夊埌鐨勭増鏈拰缂撳瓨鏂囦欢涓嶄竴鑷达紝MCP 杩炴帴鏂紑銆?
瑙ｅ喅鏂规锛歠ork 鍒?`TonyQ-AI/mimo-mcp-server`锛屼慨澶嶆簮鐮佸榻?npm 鐗堬紝浠?`tonyq-mimo-mcp-server` 鍚嶇О鍙戝竷鍒?npm锛屽畬鍏ㄨ劚绂诲 alionsss 鐨勪緷璧栥€?
**涓轰粈涔?`MIMO_API_BASE` 鏀规垚 `MIMO_API_URL`锛?*

MCP Server 婧愮爜閫氳繃 `process.env.MIMO_API_URL` 璇诲彇 API 鍦板潃锛岄粯璁ゅ€兼槸 Token Plan 绔偣锛坄token-plan-cn.xiaomimimo.com`锛夈€傚鏋滅敤鏍囧噯 API key 浣嗛厤鐨勬槸 `MIMO_API_BASE`锛屾湇鍔″櫒璇讳笉鍒拌嚜瀹氫箟鍦板潃锛岄粯璁よ蛋 Token Plan 绔偣瀵艰嚧 401銆?
**涓轰粈涔堝伐鍏峰悕鍔犱簡 `mimo_` 鍓嶇紑锛?*

npm 鍙戝竷鐗?v0.1.2 灏嗘墍鏈夊伐鍏峰悕鍔犱簡 `mimo_` 鍓嶇紑锛坄understand_image` 鈫?`mimo_understand_image`锛夛紝浣?Reasonix 缂撳瓨鏂囦欢浠嶇敤鏃у悕銆傝皟鐢ㄦ椂鍚嶅瓧涓嶅尮閰嶏紝MCP Server 杩斿洖 `read: EOF`銆備慨澶嶆柟寮忥細
1. 鏇存柊缂撳瓨鏂囦欢涓?7 涓伐鍏峰悕鍖归厤鏂板悕
2. 閲嶆柊鍙戝竷 npm 鍖呮椂淇濇寔宸ュ叿鍚嶄竴鑷?
**涓轰粈涔堥厤缃枃浠朵笉鑳界洿鎺ュ啓 key锛?*

Reasonix 鐨?`redact_tool_output` 瀹夊叏鏈哄埗浼氳嚜鍔ㄥ皢鏄剧ず涓殑瀵嗛挜鏇挎崲涓?`*`銆傚鏋滅洿鎺ュ鍒舵樉绀轰腑鐨?key锛堝甫鏄熷彿锛夊啓鍏ラ厤缃枃浠讹紝瀹為檯浼犲叆鐨勬槸甯︽槦鍙风殑鏃犳晥 key銆傚繀椤婚€氳繃鐜鍙橀噺寮曠敤锛坄${MI*******KEY}`锛夋垨浣跨敤鑴氭湰浠?`.env` 鏂囦欢璇诲彇鍐欏叆銆?
---

## 11. 鎶€鑳界洰褰?
40鎶€鑳? 缂栨帓2 + 瀛怉gent5 + 鍙傝€? + 鏋舵瀯4 + 鏂规硶7 + 瀹℃煡4 + 鎻愪氦5 + 杈呭姪3 + MCP5 + 鎵╁睍4

---

## 12. v4.x 鐗堟湰婕旇繘

### v4.1.2 (2026-07-13) 鈥?MCP 鐙珛鍙戝竷

**MCP 鏈嶅姟杩佺Щ**
- 鑷缓 npm 鍖?`tonyq-mimo-mcp-server`锛岃劚绂诲 alionsss 鐨勪緷璧?- fork 婧愮爜鍒?TonyQ-AI锛屼慨澶嶅伐鍏峰悕瀵归綈銆佹瀯寤?dist銆乥in 鍚嶇粺涓€
- GitHUb 婧愮爜浠撳簱锛歨ttps://github.com/TonyQ-AI/mimo-mcp-server

**鐜鍙橀噺淇甪ux**
- `MIMO_API_BASE` 鈫?`MIMO_API_URL`锛圡CP Server 瀹為檯璇荤殑鏄悗鑰咃級
- URL 琛ュ叏 `/chat/completions` 璺緞锛堜箣鍓嶇己璺緞瀵艰嚧 404锛?- 閰嶇疆鏂囦欢绂佹纭紪鐮?key锛屾敼鐢?`${MIMO_API_KEY}` 鐜鍙橀噺寮曠敤锛堥槻姝㈤伄缃╁啓閿欙級

**npx 鍙傛暟淇**
- bin 鍚嶄粠 `mimo-mcp-server` 鏀逛负 `tonyq-mimo-mcp-server`锛屼笌鍖呭悕涓€鑷达紝npx 鍙洿鎺ユ壘鍒板叆鍙?- 閰嶇疆绀轰緥鏇存柊涓?`args = ["-y", "tonyq-mimo-mcp-server"]`锛堟棤闇€ `--` 鍒嗛殧绗︼級

**涓€閿畨瑁?鍗囩骇**
- workflow-installer 瀹夎鍣紙`/workflow-installer`锛夋柊澧炲崌绾фā寮?- 璇淬€屽崌绾у伐浣滄祦銆嶈嚜鍔ㄦ媺鏈€鏂版妧鑳?+ 淇 MCP 閰嶇疆 + 娓呴櫎缂撳瓨

### v4.0 (2026-07-12) 鈥?鍙屽紩鎿庢灦鏋?
- 缂栨帓鍣?寮曟搸鎷嗗垎
- 纭棬鎺?Go/No-Go + 鎵弿鍥為€€)
- 鍚姩鑷
- task_plan.md缁熶竴鍩哄噯

### v3.x (2026-07-07~12) 鈥?13闃舵鎵╁睍

- 8->13闃舵銆佷氦浜掓緞娓呫€侀鍩熷缓妯°€佷氦鍙夐獙璇併€佺煡璇嗘矇娣€

### v2.x (2026-07-01~06) 鈥?鍩虹娴佹按绾?
- 8闃舵銆乀DD/璋冭瘯銆乧heckpoint杩借釜

---

> 鐗堟湰: 4.1.2 路 鏈€鍚庢洿鏂? 2026-07-13
