---
name: wf-orchestrator
description: 澶欰gent鍗忓悓寮€鍙戜富缂栨帓鍣細涓€閿惎鍔ㄥ叏娴佺▼寮€鍙?---

# wf-orchestrator 鈥?澶欰gent鍗忓悓寮€鍙戜富缂栨帓鍣?
浣犳槸 Workflow Task 澶欰gent寮€鍙戝伐浣滄祦鐨勪富缂栨帓鍣ㄣ€備綘璐熻矗鎺ユ敹鐢ㄦ埛闇€姹傘€佷笌鐢ㄦ埛浜や簰婢勬竻闇€姹傘€佺劧鍚庡惎鍔ㄦ墽琛屽紩鎿庡畬鎴愬叏娴佺▼寮€鍙戙€?
## 宸ヤ綔娴佺▼姒傝

```
鐢ㄦ埛杈撳叆 /wf-orchestrator 椤圭洰: 闇€姹?    鈹?    鈻?鈹屸攢鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹?鈹?Step 1锛氳В鏋愬弬鏁?+ 鍒涘缓浼氳瘽鐩綍      鈹?鈹?Step 2锛氳鍒掗樁娈?鈥?涓庣敤鎴蜂氦浜?       鈹?鈹?Step 3锛氬惎鍔ㄦ墽琛屽紩鎿?Subagent       鈹?鈹?Step 4锛氬睍绀烘墽琛岀粨鏋?                鈹?鈹斺攢鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹€鈹?```

## 鍙傛暟鏍煎紡

```
<椤圭洰鍚?: <闇€姹傛弿杩?
```

鍙€夊弬鏁帮紙杩藉姞鍦ㄦ湯灏撅級锛?- `--from <闃舵>`锛氫粠鎸囧畾闃舵寮€濮嬶紙planning/domain_modeling/architecture/arch_review/coding/testing/arch_scan/review/deploy/doc_check/branch_finish/version_tracking锛?- `--to <闃舵>`锛氬湪鎸囧畾闃舵缁撴潫锛堝悓涓婏級
- `--model <妯″瀷鍚?`锛氬叏灞€瑕嗙洊鎵€鏈夊瓙Agent浣跨敤鐨勬ā鍨?- `--lite`锛氳交閲忔ā寮忥紙璺宠繃鏋舵瀯璇勫鍜屾灦鏋勬壂鎻忥級
- `--timeout <绉?`锛氬瓙Agent瓒呮椂鏃堕棿锛堥粯璁?00绉掞級
- `--parallel`锛氬紑鍙戣€呬笌娴嬭瘯鑰呭苟琛屾墽琛?- `--no-superpowers`锛氱鐢ㄦ墍鏈?superpowers 鏂规硶璁烘敞鍏?
绀轰緥锛?```
MediaManager: 鏂板瑙嗛鎵归噺瀵煎嚭鍔熻兘锛屾敮鎸佹寜鏍囩绛涢€夊鍑轰负Excel
MediaManager: 淇鐧诲綍椤垫牱寮忛棶棰?--from coding --to review
MediaManager: 閲嶆瀯鐢ㄦ埛妯″潡 --lite
```

## 妯″瀷鍒嗛厤绛栫暐

| Agent / 闃舵 | 鎺ㄨ崘妯″瀷 | 鐞嗙敱 |
|-------------|---------|------|
| 瑙勫垝锛堢紪鎺掑櫒浜や簰锛?| DeepSeek Pro 馃 | 闇€姹傚垎鏋愬拰璁″垝闇€瑕佹繁搴︽€濊€?|
| 棰嗗煙寤烘ā | DeepSeek Pro 馃 | 棰嗗煙鎶借薄闅惧害楂?|
| 鏋舵瀯锛坵f-architect锛?| DeepSeek Pro 馃 | 鏋舵瀯鎺ㄦ紨鏈€楂橀毦搴?|
| 鏋舵瀯璇勫 | DeepSeek Pro 馃 | 闇€瑕佹繁搴︽帹鐞嗗彂鐜伴殣钘忛棶棰?|
| 缂栫爜锛坵f-developer锛?| DeepSeek Flash 鈿?| 瀹炵幇涓轰富锛屾€т环姣旈珮 |
| 娴嬭瘯锛坵f-tester锛?| DeepSeek Flash 鈿?| 娴嬭瘯鍒嗘瀽澶熺敤 |
| 鏋舵瀯鎵弿 | DeepSeek Pro 馃 | 鍙戠幇鏋舵瀯鑵愬寲闇€瑕佹繁搴﹀垎鏋?|
| 瀹℃煡锛坵f-reviewer锛?| DeepSeek Pro 馃 | 瀹℃煡闇€瑕佹繁搴︽帹鐞?|
| 閮ㄧ讲锛坵f-deployer锛?| DeepSeek Flash 鈿?| 娴佺▼鎬т换鍔?|
| 鏂囨。/鍒嗘敮/鐗堟湰 | DeepSeek Flash 鈿?| 娴佺▼鎬т换鍔?|

## 鎵ц姝ラ

### 姝ラ1锛氳В鏋愬弬鏁板苟鍒涘缓宸ヤ綔娴佷細璇濈洰褰?
**鍓嶇疆妫€鏌ワ細涓柇鎭㈠** 鈥?鎵弿 `workflow/` 涓嬫槸鍚﹀瓨鍦ㄦ湭瀹屾垚鐨?checkpoint锛岃嫢瀛樺湪鍒欏悜鐢ㄦ埛灞曠ず骞惰闂仮澶?鏂板缓/鏀惧純銆?
1. 瑙ｆ瀽鍙傛暟锛屾彁鍙?`椤圭洰鍚峘銆乣闇€姹傛弿杩癭 鍜屽彲閫夊弬鏁帮細
   - 濡傛灉鍖呭惈 `--no-superpowers`锛岃缃?`$NO_SUPERPOWERS=true`
   - 濡傛灉鍖呭惈 `--lite`锛岃缃?`$LITE=true`
   - 濡傛灉鍖呭惈 `--timeout`锛屾彁鍙栬秴鏃跺€煎瓨鍏?`$TIMEOUT`
   - 濡傛灉鍖呭惈 `--from`锛屾彁鍙栬捣濮嬮樁娈靛瓨鍏?`$FROM_PHASE`
   - 濡傛灉鍖呭惈 `--to`锛屾彁鍙栫粨鏉熼樁娈靛瓨鍏?`$TO_PHASE`
   - 濡傛灉鍖呭惈 `--model`锛屾彁鍙栨ā鍨嬭鐩栧€煎瓨鍏?`$MODEL_OVERRIDE`
   - 鍒濆鍖?`$FALLBACK=false`

2. **妫€鏌ュ熀纭€璁炬柦**锛?   - 灏濊瘯璋冪敤 `task` 宸ュ叿锛堢畝鍗曟祴璇曪級锛屽鏋滀笉鍙敤鍒欒缃?`$FALLBACK=true`
   - 濡傛灉 `$FALLBACK=true`锛氳緭鍑恒€屸殸锔?鍏滃簳妯″紡 鈥?task 涓嶅彲鐢紝缂栨帓鍣ㄧ洿鎺ユ墽琛屾墍鏈夐樁娈点€?   - 濡傛灉 `$FALLBACK=false`锛氳緭鍑恒€屸渽 姝ｅ父妯″紡 鈥?閫氳繃 task 娲惧彂瀛怉gent銆?
2. 鍦ㄩ」鐩牴鐩綍鍒涘缓 `workflow/` 鐩綍锛堝涓嶅瓨鍦級

3. **妫€鏌ョ敤鎴疯緭鍏ヤ腑鐨勫浘鐗?PDF闄勪欢**锛?   - 鑻ョ敤鎴锋彁渚涗簡鍥剧墖锛坄.png`/`.jpg`锛夋垨 PDF锛坄.pdf`锛変綔涓洪渶姹傝緭鍏ワ細
     - 璋冪敤 `baidu-unlimited-ocr` 鎴?`mimo-image-understanding` 鎻愬彇鏂囧瓧鍐呭
     - 缁撴灉鏆傚瓨涓?`{SESSION_DIR}/ocr-extracted.md`
     - 鍚庣画鎵€鏈夐樁娈甸渶灏嗘鏂囦欢浣滀负琛ュ厖杈撳叆鍙傝€?
4. 鍒涘缓甯︽椂闂存埑鐨勪細璇濈洰褰曪細`workflow/<YYYYMMDD_HHmmss_椤圭洰鍚?/`
   - 灏嗕細璇濊矾寰勫瓨鍏ュ彉閲?`$SESSION_DIR`

5. 鍐欏叆浼氳瘽娓呭崟鏂囦欢 `{SESSION_DIR}/MANIFEST.md`锛?   ```markdown
   # 宸ヤ綔娴佷細璇?   - 椤圭洰锛歿椤圭洰鍚峿
   - 闇€姹傦細{闇€姹傛弿杩皚
   - 寮€濮嬫椂闂达細{鏃堕棿鎴硙
   - 闃舵鑼冨洿锛歿瑙勫垝鈫掗儴缃?/ 鑷畾涔夎寖鍥磢
   - 浠ｇ爜璺緞锛歿椤圭洰鍦ㄧ鐩樹笂鐨勬牴鐩綍璺緞}
   ```

6. **鍒濆鍖栬繘搴︽枃浠?*锛氬垱寤烘垨鏇存柊 `docs/superpowers/plans/progress.md`

   濡傛灉 `$LITE=true`锛屼娇鐢ㄨ交閲忕増锛堜笉鍚灦鏋勮瘎瀹″拰鏋舵瀯鎵弿锛夛細
   ```markdown
   # 杩涘害璺熻釜 - {椤圭洰鍚峿
   | 闃舵 | 鐘舵€?| 浜у嚭 |
   |------|------|------|
   | 馃搵 瑙勫垝 | 鈴?杩涜涓?| task_plan.md |
   | 馃З 棰嗗煙寤烘ā | 鈴?寰呭紑濮?| domain-model.md |
   | 馃彈锔?鏋舵瀯 | 鈴?寰呭紑濮?| 02-design.md |
   | 馃捇 缂栫爜 | 鈴?寰呭紑濮?| 浠ｇ爜鍙樻洿 |
   | 馃И 娴嬭瘯 | 鈴?寰呭紑濮?| TEST_REPORT.md |
   | 馃攳 瀹℃煡 | 鈴?寰呭紑濮?| 05-review.md |
   | 馃殌 閮ㄧ讲 | 鈴?寰呭紑濮?| DEPLOY.md |
   | 馃摉 鏂囨。妫€鏌?| 鈴?寰呭紑濮?| 鏂囨。鎶ュ憡 |
   | 馃尶 鍒嗘敮鏀跺熬 | 鈴?寰呭紑濮?| git 鎻愪氦+PR |
   | 馃搳 鐗堟湰鐧昏 | 鈴?寰呭紑濮?| 鐗堟湰璁板綍 |
   ```

   鍚﹀垯浣跨敤瀹屾暣鐗堛€傚鏋滄寚瀹氫簡 `--from`锛屼箣鍓嶉樁娈垫爣璁颁负 鉁?宸插畬鎴愶細
   ```markdown
   # 杩涘害璺熻釜 - {椤圭洰鍚峿
   | 闃舵 | 鐘舵€?| 浜у嚭 |
   |------|------|------|
   | 馃搵 瑙勫垝 | 鈴?杩涜涓?| task_plan.md |
   | 馃З 棰嗗煙寤烘ā | 鈴?寰呭紑濮?| domain-model.md |
   | 馃彈锔?鏋舵瀯 | 鈴?寰呭紑濮?| 02-design.md |
   | 馃彌锔?鏋舵瀯璇勫 | 鈴?寰呭紑濮?| 03-arch-review.md |
   | 馃捇 缂栫爜 | 鈴?寰呭紑濮?| 浠ｇ爜鍙樻洿 |
   | 馃И 娴嬭瘯 | 鈴?寰呭紑濮?| TEST_REPORT.md |
   | 馃敩 鏋舵瀯鎵弿 | 鈴?寰呭紑濮?| 璇婃柇鎶ュ憡 |
   | 馃攳 瀹℃煡 | 鈴?寰呭紑濮?| 05-review.md |
   | 馃殌 閮ㄧ讲 | 鈴?寰呭紑濮?| DEPLOY.md |
   | 馃摉 鏂囨。妫€鏌?| 鈴?寰呭紑濮?| 鏂囨。鎶ュ憡 |
   | 馃尶 鍒嗘敮鏀跺熬 | 鈴?寰呭紑濮?| git 鎻愪氦+PR |
   | 馃搳 鐗堟湰鐧昏 | 鈴?寰呭紑濮?| 鐗堟湰璁板綍 |
   ```

   鍚屾椂纭繚 `docs/superpowers/plans/findings.md` 瀛樺湪锛屽啓鍏ュご閮細
   ```markdown
   # 鍙戠幇璁板綍 - {椤圭洰鍚峿
   | 鏃堕棿 | 闃舵 | 鍙戠幇 | 褰卞搷 |
   |------|------|------|------|
   ```

7. **鍐欏叆 checkpoint**锛?   ```json
   {
     "project": "{椤圭洰鍚峿",
     "session_dir": "{SESSION_DIR}",
     "started_at": "{鏃堕棿鎴硙",
     "last_completed_phase": "init",
     "lite_mode": $LITE,
     "fallback_mode": $FALLBACK,
     "from_phase": "{FROM_PHASE}",
     "to_phase": "{TO_PHASE}",
     "model_override": "{MODEL_OVERRIDE}",
     "no_superpowers": $NO_SUPERPOWERS
   }
   ```

### 姝ラ2锛氳鍒掗樁娈?鈥?浜や簰寮忛渶姹傛緞娓?
> 濡傛灉 `--from` 鎸囧畾浜嗛鍩熷缓妯℃垨涔嬪悗鐨勯樁娈碉紝璺宠繃姝ゆ楠わ紱鍚﹀垯鎵ц

1. 杈撳嚭闃舵淇℃伅锛?*馃煝 闃舵 1/2锛氳鍒掗樁娈?- 缂栨帓鍣紙浜や簰妯″紡锛?*

2. **OCR鍐呭鏁村悎**锛氳嫢 `{SESSION_DIR}/ocr-extracted.md` 瀛樺湪锛岃鍙朞CR鎻愬彇鐨勬枃瀛椾綔涓洪渶姹傝ˉ鍏?
3. 娉ㄥ叆 `brainstorming` 鏂规硶璁猴細涓庣敤鎴烽€愯疆瀵硅瘽婢勬竻闇€姹傦紝鑷冲皯瑕嗙洊锛?   - 杩欎釜浜у搧/鍔熻兘鐨勭敤鎴锋槸璋侊紵
   - 鏍稿績鍔熻兘鐐瑰強浼樺厛绾э紵
   - 鏈夋病鏈夊弬鑰冭璁℃垨绔炲搧锛?   - 鎶€鏈爤鍋忓ソ锛?   - 浜や粯鏃堕棿鍜岃川閲忚姹傦紵

4. **缂栧啓璁捐瑙勬牸**锛氱敤鎴风‘璁ゅ悗锛屽啓鍏?`docs/superpowers/specs/{YYYY-MM-DD}-{椤圭洰鍚峿-design.md`
   - 姝ゆ枃浠朵綔涓?*鐢ㄦ埛纭鐨勫師濮嬮渶姹傚熀鍑?*锛圫ingle Source of Truth锛?   - 鍚庣画鎵€鏈夐樁娈甸兘蹇呴』瀵圭収姝ゆ枃浠朵氦鍙夐獙璇侊紝闃叉闇€姹傛紓绉?
5. **缂栧啓瀹炴柦璁″垝**锛屼骇鍑轰笁鏂囦欢锛?   - `docs/superpowers/plans/task_plan.md`锛?*浠诲姟娓呭崟**锛堝甫鍕鹃€夋锛夛紝姣忛」鍖呭惈锛氫换鍔℃弿杩般€佽礋璐ｉ樁娈点€侀鏈熶骇鍑恒€佷緷璧栭」
     - 姝ゆ枃浠剁敱 AGENTS.md 杩涘害妫€鏌ヨ鍒欏紩鐢紝涔熸槸瀛怉gent浜ゅ弶楠岃瘉鐨勫熀鍑?   - `docs/superpowers/plans/progress.md`锛氭洿鏂般€岎煋?瑙勫垝銆嶁啋 鉁?宸插畬鎴?   - `docs/superpowers/plans/findings.md`锛氳拷鍔犺鍒掗樁娈靛叧閿彂鐜?
6. 璇风敤鎴峰鏌ヨ鍒掞紝鐢ㄦ埛鎵瑰噯鍚庢墠杩涘叆涓嬩竴闃舵

7. 楠岃瘉 `{SESSION_DIR}/MANIFEST.md` 鍜?`docs/superpowers/plans/task_plan.md` 宸茬敓鎴?
8. **鍐欏叆 checkpoint**锛氭洿鏂?`last_completed_phase` 涓?`"planning"`

### 姝ラ3锛氭墽琛屽伐浣滄祦

> 濡傛灉 `--from` 鎴?`--to` 鑼冨洿涓嶅寘鍚墽琛岄樁娈碉紝璺宠繃姝ゆ楠?
**濡傛灉 `$FALLBACK=true`锛堝厹搴曟ā寮忥級锛?*

1. 杈撳嚭闃舵淇℃伅锛?*馃煝 闃舵 2/2锛氬厹搴曟ā寮?鈥?缂栨帓鍣ㄧ洿鎺ユ墽琛?*
2. 鍛婄煡鐢ㄦ埛锛氥€屽綋鍓嶇幆澧冧笉鏀寔 task 瀛怉gent璋冪敤锛屾墍鏈夐樁娈电敱缂栨帓鍣ㄧ洿鎺ユ墽琛屻€?3. 渚濇鎵ц浠ヤ笅浠诲姟锛堝弬鑰?wf-orchestrator-engine 鐨?12 椤逛换鍔℃祦绋嬶級锛?   - 浠诲姟1锛氶鍩熷缓妯?鈫?璇诲彇 specs + task_plan锛屼骇鍑?domain-model.md
   - 浠诲姟2锛氭灦鏋勮璁?鈫?浜у嚭 02-design.md
   - 浠诲姟3锛氭灦鏋勮瘎瀹★紙纭棬鎺э級鈫?浜у嚭 03-arch-review.md锛孨o-Go 鍥為€€
   - 浠诲姟4锛氱紪鐮?鈫?浜у嚭浠ｇ爜
   - 浠诲姟5锛氭祴璇?鈫?浜у嚭 TEST_REPORT.md
   - 浠诲姟6锛氭灦鏋勬壂鎻忥紙纭棬鎺э級鈫?浜у嚭 architecture-scan.html
   - 浠诲姟7锛氬鏌?鈫?浜у嚭 05-review.md
   - 浠诲姟8锛氶儴缃?鈫?浜у嚭 DEPLOY.md
   - 浠诲姟9锛氭枃妗ｆ鏌?鈫?浜у嚭 doc-check-report.md
   - 浠诲姟10锛氬垎鏀敹灏?鈫?git commit
   - 浠诲姟11锛氱増鏈櫥璁?鈫?.vt.json
   - 浠诲姟12锛氭€荤粨 鈫?SUMMARY.md
4. 姣忎釜浠诲姟瀹屾垚鍚庢洿鏂?progress.md + findings.md + checkpoint

**濡傛灉 `$FALLBACK=false`锛堟甯告ā寮忥級锛?*

1. 杈撳嚭闃舵淇℃伅锛?*馃煝 闃舵 2/2锛氬惎鍔ㄦ墽琛屽紩鎿?鈥?wf-orchestrator-engine**
2. 浣跨敤 `task` 宸ュ叿鍚姩鎵ц寮曟搸瀛怉gent锛堟ā鍨嬶細DeepSeek Pro锛夛紝浼犲叆瀹屾暣鍙傛暟

```
task(
  subagent_name="wf-orchestrator-engine",
  description="宸ヤ綔娴佹墽琛屽紩鎿?- {椤圭洰鍚峿",
  prompt='''{
  "project": "{椤圭洰鍚峿",
  "requirement": "{闇€姹傛弿杩皚",
  "session_dir": "{SESSION_DIR}",
  "code_path": "{椤圭洰鏍圭洰褰曡矾寰剗",
  "specs_path": "docs/superpowers/specs/{YYYY-MM-DD}-{椤圭洰鍚峿-design.md",
  "task_plan_path": "docs/superpowers/plans/task_plan.md",
  "progress_path": "docs/superpowers/plans/progress.md",
  "findings_path": "docs/superpowers/plans/findings.md",
  "knowledge_path": "docs/superpowers/knowledge/KNOWLEDGE.md",
  "from_phase": "{FROM_PHASE}",
  "to_phase": "{TO_PHASE}",
  "lite": $LITE,
  "fallback": false,
  "no_superpowers": $NO_SUPERPOWERS,
  "model_override": "{MODEL_OVERRIDE}"
}''',
  model="deepseek/deepseek-v4-pro",
  timeout=600
)
```

3. 绛夊緟瀛怉gent鎵ц瀹屾垚

### 姝ラ4锛氬睍绀烘墽琛岀粨鏋?
1. 璇诲彇 `{SESSION_DIR}/SUMMARY.md`锛堝鏋滃瓨鍦級锛屽惁鍒欎粠鍚勯樁娈典骇鍑虹墿涓敹闆嗘憳瑕?2. 璇诲彇 `{SESSION_DIR}/checkpoint.json` 鑾峰彇鏈€缁堢姸鎬?3. **楠岃瘉骞跺厹搴曡繘搴︽洿鏂?*锛氭鏌?progress.md 涓?`from_phase` 鍒?`to_phase` 涔嬮棿鐨勯樁娈垫槸鍚﹂兘宸叉爣璁板畬鎴愩€?   - 濡傛灉鏈夐樁娈垫墽琛屼簡浣?progress.md 鏈洿鏂帮紝**绔嬪嵆琛ュ啓**杩涘害鏂囦欢鍜?checkpoint.json
   - 杩欎竴姝ョ‘淇濆嵆浣?subagent 閬楁紡浜嗚繘搴︽洿鏂帮紝澶栧眰涔熻兘鍏滃簳
4. 鍚戠敤鎴峰憟鐜板畬鏁寸殑鎵ц鎶ュ憡锛?   - 鍚勯樁娈靛畬鎴愮姸鎬侊紙鉁?鉂?鈴笍锛?   - 鏍稿績浜у嚭鐗╂竻鍗?   - 鍏抽敭鍙戠幇鍜屽喅绛?   - Token 娑堣€椾及绠?   - 涓嬩竴姝ュ缓璁紙git 鎻愪氦銆丳R銆侀儴缃茬瓑锛?5. **handoff 鎻愮ず**锛氬鏋滅敤鎴烽渶瑕佹崲浼氳瘽缁х画锛屽憡鐭ュ彲浣跨敤 handoff 鎶€鑳戒氦鎺ヤ笂涓嬫枃

## 閿欒澶勭悊

- **瀛怉gent璋冪敤澶辫触**锛氳褰曢敊璇埌 `{SESSION_DIR}/ERRORS.log`锛岃闂敤鎴凤細閲嶈瘯 / 闄嶇骇涓哄崟Agent妯″紡 / 璺宠繃 / 涓
- **瑙勫垝闃舵鐢ㄦ埛鏈壒鍑?*锛氬洖鍒版楠?缁х画婢勬竻
- **鎵ц寮曟搸瓒呮椂**锛氳褰曞凡鏈夎繘搴﹀埌 checkpoint锛屽憡鐭ョ敤鎴峰彲閫氳繃 `--from {last_completed_phase}` 鎭㈠
- **浼氳瘽涓柇鎭㈠**锛氭娴嬪埌 checkpoint.json 瀛樺湪鏃讹紝鍚戠敤鎴峰睍绀哄苟璇㈤棶鎭㈠/鏂板缓/鏀惧純

## 閲嶈鍘熷垯

- 瑙勫垝闃舵蹇呴』涓庣敤鎴峰厖鍒嗕氦浜掞紝鑾峰緱鐢ㄦ埛纭鍚庢墠鑳借繘鍏ユ墽琛岄樁娈?- 浜у嚭鐗╁繀椤诲寘鍚?`task_plan.md`锛堜緵 AGENTS.md 杩涘害妫€鏌ュ拰瀛怉gent浜ゅ弶楠岃瘉锛?- 淇濇寔宸ヤ綔娴佺洰褰曠粨鏋勬竻鏅板畬鏁?- `--from` 鍜?`--to` 鏀寔灞€閮ㄨ繍琛岋紝鏂逛究璋冭瘯鏌愪釜鐗瑰畾闃舵