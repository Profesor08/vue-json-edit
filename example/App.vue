<template>
	<div class="j-w">
		<h1 class="t">Vue-Json-Edit</h1>
		<div class="editor-w clearfix">
			<div class="w-2">
				<div class="editor">
					<input type="text" v-model="search">
					<JsonEditor
						:options="{
							confirmText: 'confirm',
							cancelText: 'cancel',
						}"
						:objData="jsonData" 
						:search="search"
						v-model="jsonData" ></JsonEditor>
				</div>
			</div>
			<div class="w-2">
				<div class="code-pre">
					<div slot="content">
						<pre>
							<code class="json" id="res_code"></code>
						</pre>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import hljs from 'highlight.js'

export default {
	name: 'app',
	data: function() {
		return {
			search: "",
			
			jsonData: {
        resets_column: "ResetCount",
        grand_resets_column: "MasterResetCount",
        cahce_global: false,
        currency: {
          wcoinc: {
            table: "CashShopData",
            column: "WCoinC",
            account: "AccountID",
          },
          wcoinp: {
            table: "CashShopData",
            column: "WCoinP",
            account: "AccountID",
          },
          goblin_point: {
            table: "CashShopData",
            column: "GoblinPoint",
            account: "AccountID",
          },
        },
        gens: {
          family: {
            table: "Gens_Rank",
            column: "Family",
            contribution: "Contribution",
            character: "Name",
          },
          rank: { table: "Gens_Reward", column: "Rank", character: "Name" },
        },
        crywolf: {
          table: "WZ_CW_INFO",
          occupy: "CRYWOLF_OCCUFY",
          state: "CRYWOLF_STATE",
        },
        events: {
          regular: [
            [
              "BloodCastle",
              0,
              "01:00",
              "03:00",
              "05:00",
              "07:00",
              "09:00",
              "11:00",
              "13:00",
              "15:00",
              "17:00",
              "19:00",
              "21:00",
              "23:00",
            ],
            [
              "DevilSquare",
              0,
              "00:00",
              "02:00",
              "04:00",
              "06:00",
              "08:00",
              "10:00",
              "12:00",
              "14:00",
              "16:00",
              "18:00",
              "20:00",
              "22:00",
            ],
            [
              "Chaos Castle",
              0,
              "00:30",
              "02:30",
              "06:30",
              "10:30",
              "18:30",
              "22:30",
            ],
            ["Happy Hour", 0, "08:30", "20:30"],
            ["Lorencia Battle", 0, "19:30"],
            ["Drop Event", 0, "10:00", "20:00"],
            ["Happy Hour", 0, "19:00"],
            ["Question Answer", 0, "17:30", "21:30"],
            ["Search Event", 0, "04:30", "16:30"],
            [
              "White Wizard",
              0,
              "00:40",
              "03:40",
              "06:40",
              "09:40",
              "12:40",
              "15:40",
              "18:40",
              "21:40",
            ],
          ],
          bosses: [
            [
              "Golden Invasion",
              0,
              "09:48",
              "11:10",
              "15:30",
              "16:25",
              "20:25",
              "21:10",
            ],
            [
              "Kundum",
              1,
              "01:00",
              "03:00",
              "05:00",
              "07:00",
              "09:00",
              "11:00",
              "13:00",
              "15:00",
              "17:00",
              "19:00",
              "21:00",
              "23:00",
            ],
            [
              "Medusa",
              1,
              "00:00",
              "02:00",
              "04:00",
              "06:00",
              "08:00",
              "10:00",
              "12:00",
              "14:00",
              "16:00",
              "18:00",
              "20:00",
              "22:00",
            ],
            ["Erohim", 1, "00:30", "02:30", "06:30", "10:30", "18:30", "22:30"],
            [
              "White Wizard",
              1,
              "09:48",
              "11:10",
              "15:30",
              "16:25",
              "20:25",
              "21:10",
            ],
            ["Selupan", 1, "08:30", "20:30"],
            ["Nightmare", 1, "19:30"],
            ["Red Dragon", 1, "10:00", "20:00"],
            ["Skeleton King", 1, "19:00"],
          ],
        },
      }
		}
	},
	watch: {
		'jsonData': function () {
			let c = this.formatJson(JSON.stringify(this.jsonData))
			this.drawResCode(c)
		}
	},
	methods: {
		//JSON format print
		formatJson: function(txt, compress /*是否为压缩模式*/) {
			/* 格式化JSON源码(对象转换为JSON文本) */
			var indentChar = "  ";
			if (/^\s*$/.test(txt)) {
				console.error("数据为空,无法格式化! ");
				return;
			}
			try {
				var data = eval("(" + txt + ")");
			} catch (e) {
				throw ("数据源语法错误,格式化失败! 错误信息: " + e.description, "err");
				return;
			}
			var draw = [],
				last = false,
				This = this,
				line = compress ? "" : "\n",
				nodeCount = 0,
				maxDepth = 0;

			var notify = function(name, value, isLast, indent /*缩进*/, formObj) {
				nodeCount++; /*节点计数*/
				for (var i = 0, tab = ""; i < indent; i++) tab += indentChar; /* 缩进HTML */
				tab = compress ? "" : tab; /*压缩模式忽略缩进*/
				maxDepth = ++indent; /*缩进递增并记录*/
				if (value && value.constructor == Array) {
					/*处理数组*/
					draw.push(
						tab + (formObj ? '"' + name + '":' : "") + "[" + line
					); /*缩进'[' 然后换行*/
					for (var i = 0; i < value.length; i++)
						notify(i, value[i], i == value.length - 1, indent, false);
					draw.push(
						tab + "]" + (isLast ? line : "," + line)
					); /*缩进']'换行,若非尾元素则添加逗号*/
				} else if (value && typeof value == "object") {
					/*处理对象*/
					draw.push(
						tab + (formObj ? '"' + name + '":' : "") + "{" + line
					); /*缩进'{' 然后换行*/
					var len = 0,
						i = 0;
					for (var key in value) len++;
					for (var key in value) notify(key, value[key], ++i == len, indent, true);
					draw.push(
						tab + "}" + (isLast ? line : "," + line)
					); /*缩进'}'换行,若非尾元素则添加逗号*/
				} else {
					if (typeof value == "string") value = '"' + value + '"';
					draw.push(
						tab +
						(formObj ? '"' + name + '":' : "") +
						value +
						(isLast ? "" : ",") +
						line
					);
				}
			};
			var isLast = true,
				indent = 0;
			notify("", data, isLast, indent, false);
			return draw.join("");
		},

		//绘制res body
		drawResCode: function (content) {
			var target = document.getElementById('res_code');
			target.textContent = content
			hljs.highlightBlock(target)
		},
	},
	mounted: function() {
		let c = this.formatJson(JSON.stringify(this.jsonData))
		this.drawResCode(c)
	}
}
</script>

<style>
@import url('../node_modules/highlight.js/styles/github.css');

</style>


