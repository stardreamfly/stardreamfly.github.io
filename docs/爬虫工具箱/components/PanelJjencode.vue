<template>
	<div>
		<h3>{{ panelTitle }}</h3>
		<div class="input-container">
			<el-input v-model="inputText" style="width: 100%" :rows="6" type="textarea" placeholder="请输入要编码/解码的内容" />
		</div>
		<div>
			<div>
				全局变量名:<el-input v-model="globalValue" style="width: 240px" placeholder="全局变量名" />
			</div>
			<div style="display: flex;align-items: center;border: 1px solid blue;padding-left: 5px;">
				<el-checkbox disabled v-model="checked">回文（顺读和倒读都一样）【有bug，先禁用】</el-checkbox>
				<p>Value: {{ checked }}</p>
			</div>
			<el-button type="primary" @click="encodeURL">编码</el-button>
			<el-button type="primary" @click="decodeURL">解码</el-button>
			<div>编码无问题，汉字编码后解码可能会出错</div>
		</div>
		<el-input v-model="outputText" style="width: 100%" :rows="8" type="textarea" placeholder="编码/解码后的文字" />
	</div>
</template>

<script setup>
import { ref } from 'vue';
import {ElInput,ElCheckbox,ElButton} from 'element-plus'

const panelTitle = ref('JJencode 编码解码');
const inputText = ref('alert("您好啊，我是Python斗罗~")');
const outputText = ref('');
const checked = ref(false)
const globalValue = ref('Python斗罗')
function jjencode(gv, text) {
	var r = "";
	var n;
	var t;
	var b = ["___", "__$", "_$_", "_$$", "$__", "$_$", "$$_", "$$$", "$___", "$__$", "$_$_", "$_$$", "$$__", "$$_$", "$$$_", "$$$$",];
	var s = "";
	for (var i = 0; i < text.length; i++) {
		n = text.charCodeAt(i);
		if (n == 0x22 || n == 0x5c) {
			s += "\\\\\\" + text.charAt(i).toString(16);
		} else if ((0x21 <= n && n <= 0x2f) || (0x3A <= n && n <= 0x40) || (0x5b <= n && n <= 0x60) || (0x7b <= n && n <= 0x7f)) {
			//}else if( (0x20 <= n && n <= 0x2f) || (0x3A <= n == 0x40) || ( 0x5b <= n && n <= 0x60 ) || ( 0x7b <= n && n <= 0x7f ) ){
			s += text.charAt(i);
		} else if ((0x30 <= n && n <= 0x39) || (0x61 <= n && n <= 0x66)) {
			if (s) r += "\"" + s + "\"+";
			r += gv + "." + b[n < 0x40 ? n - 0x30 : n - 0x57] + "+";
			s = "";
		} else if (n == 0x6c) { // 'l'
			if (s) r += "\"" + s + "\"+";
			r += "(![]+\"\")[" + gv + "._$_]+";
			s = "";
		} else if (n == 0x6f) { // 'o'
			if (s) r += "\"" + s + "\"+";
			r += gv + "._$+";
			s = "";
		} else if (n == 0x74) { // 'u'
			if (s) r += "\"" + s + "\"+";
			r += gv + ".__+";
			s = "";
		} else if (n == 0x75) { // 'u'
			if (s) r += "\"" + s + "\"+";
			r += gv + "._+";
			s = "";
		} else if (n < 128) {
			if (s) r += "\"" + s;
			else r += "\"";
			r += "\\\\\"+" + n.toString(8).replace(/[0-7]/g, function (c) { return gv + "." + b[c] + "+" });
			s = "";
		} else {
			if (s) r += "\"" + s;
			else r += "\"";
			r += "\\\\\"+" + gv + "._+" + n.toString(16).replace(/[0-9a-f]/gi, function (c) { return gv + "." + b[parseInt(c, 16)] + "+" });
			s = "";
		}
	}
	if (s) r += "\"" + s + "\"+";

	r =
		gv + "=~[];" +
		gv + "={___:++" + gv + ",$$$$:(![]+\"\")[" + gv + "],__$:++" + gv + ",$_$_:(![]+\"\")[" + gv + "],_$_:++" +
		gv + ",$_$$:({}+\"\")[" + gv + "],$$_$:(" + gv + "[" + gv + "]+\"\")[" + gv + "],_$$:++" + gv + ",$$$_:(!\"\"+\"\")[" +
		gv + "],$__:++" + gv + ",$_$:++" + gv + ",$$__:({}+\"\")[" + gv + "],$$_:++" + gv + ",$$$:++" + gv + ",$___:++" + gv + ",$__$:++" + gv + "};" +
		gv + ".$_=" +
		"(" + gv + ".$_=" + gv + "+\"\")[" + gv + ".$_$]+" +
		"(" + gv + "._$=" + gv + ".$_[" + gv + ".__$])+" +
		"(" + gv + ".$$=(" + gv + ".$+\"\")[" + gv + ".__$])+" +
		"((!" + gv + ")+\"\")[" + gv + "._$$]+" +
		"(" + gv + ".__=" + gv + ".$_[" + gv + ".$$_])+" +
		"(" + gv + ".$=(!\"\"+\"\")[" + gv + ".__$])+" +
		"(" + gv + "._=(!\"\"+\"\")[" + gv + "._$_])+" +
		gv + ".$_[" + gv + ".$_$]+" +
		gv + ".__+" +
		gv + "._$+" +
		gv + ".$;" +
		gv + ".$$=" +
		gv + ".$+" +
		"(!\"\"+\"\")[" + gv + "._$$]+" +
		gv + ".__+" +
		gv + "._+" +
		gv + ".$+" +
		gv + ".$$;" +
		gv + ".$=(" + gv + ".___)[" + gv + ".$_][" + gv + ".$_];" +
		gv + ".$(" + gv + ".$(" + gv + ".$$+\"\\\"\"+" + r + "\"\\\"\")())();";

	return r;
}
function cls() {
	outputText.value = '';
}

function out(c) {
	var d = outputText.value;
	outputText.value = d + c;
}

function jjdecode(t) {
	cls();
	//clean it
	t.replace(/^\s+|\s+$/g, "");

	var startpos;
	var endpos;
	var gv;
	var gvl;
	var j,k;

	if (t.indexOf("\"\'\\\"+\'+\",") == 0) //palindrome check
	{
		//locate jjcode
		startpos = t.indexOf('$$+"\\""+') + 8;
		endpos = t.indexOf('"\\"")())()');

		//get gv
		gv = t.substring((t.indexOf('"\'\\"+\'+",') + 9), t.indexOf("=~[]"));
		gvl = gv.length;
	}
	else {
		//get gv
		gv = t.substr(0, t.indexOf("="));
		gvl = gv.length;

		//locate jjcode
		startpos = t.indexOf('"\\""+') + 5;
		endpos = t.indexOf('"\\"")())()');
	}

	if (startpos == endpos) {
		alert("No data !");
		return;
	}

	//start decoding
	var data = t.substring(startpos, endpos);

	//hex decode string
	var b = ["___+", "__$+", "_$_+", "_$$+", "$__+", "$_$+", "$$_+", "$$$+", "$___+", "$__$+", "$_$_+", "$_$$+", "$$__+", "$$_$+", "$$$_+", "$$$$+"];

	//lotu
	var str_l = "(![]+\"\")[" + gv + "._$_]+";
	var str_o = gv + "._$+";
	var str_t = gv + ".__+";
	var str_u = gv + "._+";

	//0123456789abcdef
	var str_hex = gv + ".";

	//s
	var str_s = '"';
	var gvsig = gv + ".";

	var str_quote = '\\\\\\"';
	var str_slash = '\\\\\\\\';

	var str_lower = "\\\\\"+";
	var str_upper = "\\\\\"+" + gv + "._+";

	var str_end = '"+'; //end of s loop



	while (data != "") {
		//l o t u
		if (0 == data.indexOf(str_l)) {
			data = data.substr(str_l.length);
			out("l");
			continue;
		}
		else if (0 == data.indexOf(str_o)) {
			data = data.substr(str_o.length);
			out("o");
			continue;
		}
		else if (0 == data.indexOf(str_t)) {
			data = data.substr(str_t.length);
			out("t");
			continue;
		}
		else if (0 == data.indexOf(str_u)) {
			data = data.substr(str_u.length);
			out("u");
			continue;
		}

		//0123456789abcdef
		if (0 == data.indexOf(str_hex)) {
			data = data.substr(str_hex.length);

			//check every element of hex decode string for a match 
			var i = 0;
			for (i = 0; i < b.length; i++) {
				if (0 == data.indexOf(b[i])) {
					data = data.substr((b[i]).length);
					out(i.toString(16));
					break;
				}
			}
			continue;
		}

		//start of s block
		if (0 == data.indexOf(str_s)) {
			data = data.substr(str_s.length);

			//check if "R
			if (0 == data.indexOf(str_upper)) // r4 n >= 128
			{
				data = data.substr(str_upper.length); //skip sig

				var ch_str = "";
				for (j = 0; j < 2; j++) //shouldn't be more than 2 hex chars
				{
					//gv + "."+b[ c ]				
					if (0 == data.indexOf(gvsig)) {
						data = data.substr(gvsig.length); //skip gvsig	

						for (k = 0; k < b.length; k++)	//for every entry in b
						{
							if (0 == data.indexOf(b[k])) {
								data = data.substr(b[k].length);
								ch_str += k.toString(16) + "";
								break;
							}
						}
					}
					else {
						break; //done
					}
				}

				out(String.fromCharCode(parseInt(ch_str, 16)));
				continue;
			}
			else if (0 == data.indexOf(str_lower)) //r3 check if "R // n < 128
			{
				data = data.substr(str_lower.length); //skip sig

				var ch_str = "";
				var ch_lotux = ""
				var temp = "";
				var b_checkR1 = 0;
				for (j = 0; j < 3; j++) //shouldn't be more than 3 octal chars
				{

					if (j > 1) //lotu check
					{
						if (0 == data.indexOf(str_l)) {
							data = data.substr(str_l.length);
							ch_lotux = "l";
							break;
						}
						else if (0 == data.indexOf(str_o)) {
							data = data.substr(str_o.length);
							ch_lotux = "o";
							break;
						}
						else if (0 == data.indexOf(str_t)) {
							data = data.substr(str_t.length);
							ch_lotux = "t";
							break;
						}
						else if (0 == data.indexOf(str_u)) {
							data = data.substr(str_u.length);
							ch_lotux = "u";
							break;
						}
					}

					//gv + "."+b[ c ]							
					if (0 == data.indexOf(gvsig)) {
						temp = data.substr(gvsig.length);
						for (k = 0; k < 8; k++)	//for every entry in b octal
						{
							if (0 == temp.indexOf(b[k])) {
								if (parseInt(ch_str + k + "", 8) > 128) {
									b_checkR1 = 1;
									break;
								}

								ch_str += k + "";
								data = data.substr(gvsig.length); //skip gvsig
								data = data.substr(b[k].length);
								break;
							}
						}

						if (1 == b_checkR1) {
							if (0 == data.indexOf(str_hex)) //0123456789abcdef
							{
								data = data.substr(str_hex.length);

								//check every element of hex decode string for a match 
								var i = 0;
								for (i = 0; i < b.length; i++) {
									if (0 == data.indexOf(b[i])) {
										data = data.substr((b[i]).length);
										ch_lotux = i.toString(16);
										break;
									}
								}

								break;
							}
						}
					}
					else {
						break; //done
					}
				}

				out(String.fromCharCode(parseInt(ch_str, 8)) + ch_lotux);
				continue; //step out of the while loop
			}
			else //"S ----> "SR or "S+
			{

				// if there is, loop s until R 0r +
				// if there is no matching s block, throw error

				var match = 0;
				var n;

				//searching for mathcing pure s block
				while (true) {
					n = data.charCodeAt(0);
					if (0 == data.indexOf(str_quote)) {
						data = data.substr(str_quote.length);
						out('"');
						match += 1;
						continue;
					}
					else if (0 == data.indexOf(str_slash)) {
						data = data.substr(str_slash.length);
						out('\\');
						match += 1;
						continue;
					}
					else if (0 == data.indexOf(str_end))	//reached end off S block ? +
					{
						if (match == 0) {
							alert("+ no match S block: " + data);
							return;
						}
						data = data.substr(str_end.length);

						break; //step out of the while loop
					}
					else if (0 == data.indexOf(str_upper)) //r4 reached end off S block ? - check if "R n >= 128
					{
						if (match == 0) {
							alert("no match S block n>128: " + data);
							return;
						}

						data = data.substr(str_upper.length); //skip sig

						var ch_str = "";
						var ch_lotux = "";
						for (j = 0; j < 10; j++) //shouldn't be more than 10 hex chars
						{

							if (j > 1) //lotu check
							{
								if (0 == data.indexOf(str_l)) {
									data = data.substr(str_l.length);
									ch_lotux = "l";
									break;
								}
								else if (0 == data.indexOf(str_o)) {
									data = data.substr(str_o.length);
									ch_lotux = "o";
									break;
								}
								else if (0 == data.indexOf(str_t)) {
									data = data.substr(str_t.length);
									ch_lotux = "t";
									break;
								}
								else if (0 == data.indexOf(str_u)) {
									data = data.substr(str_u.length);
									ch_lotux = "u";
									break;
								}
							}

							//gv + "."+b[ c ]				
							if (0 == data.indexOf(gvsig)) {
								data = data.substr(gvsig.length); //skip gvsig

								for (k = 0; k < b.length; k++)	//for every entry in b
								{
									if (0 == data.indexOf(b[k])) {
										data = data.substr(b[k].length);
										ch_str += k.toString(16) + "";
										break;
									}
								}
							}
							else {
								break; //done
							}
						}

						out(String.fromCharCode(parseInt(ch_str, 16)));
						break; //step out of the while loop
					}
					else if (0 == data.indexOf(str_lower)) //r3 check if "R // n < 128
					{
						if (match == 0) {
							alert("no match S block n<128: " + data);
							return;
						}

						data = data.substr(str_lower.length); //skip sig

						var ch_str = "";
						var ch_lotux = ""
						var temp = "";
						var b_checkR1 = 0;
						for (j = 0; j < 3; j++) //shouldn't be more than 3 octal chars
						{

							if (j > 1) //lotu check
							{
								if (0 == data.indexOf(str_l)) {
									data = data.substr(str_l.length);
									ch_lotux = "l";
									break;
								}
								else if (0 == data.indexOf(str_o)) {
									data = data.substr(str_o.length);
									ch_lotux = "o";
									break;
								}
								else if (0 == data.indexOf(str_t)) {
									data = data.substr(str_t.length);
									ch_lotux = "t";
									break;
								}
								else if (0 == data.indexOf(str_u)) {
									data = data.substr(str_u.length);
									ch_lotux = "u";
									break;
								}
							}

							//gv + "."+b[ c ]							
							if (0 == data.indexOf(gvsig)) {
								temp = data.substr(gvsig.length);
								for (k = 0; k < 8; k++)	//for every entry in b octal
								{
									if (0 == temp.indexOf(b[k])) {
										if (parseInt(ch_str + k + "", 8) > 128) {
											b_checkR1 = 1;
											break;
										}

										ch_str += k + "";
										data = data.substr(gvsig.length); //skip gvsig
										data = data.substr(b[k].length);
										break;
									}
								}

								if (1 == b_checkR1) {
									if (0 == data.indexOf(str_hex)) //0123456789abcdef
									{
										data = data.substr(str_hex.length);

										//check every element of hex decode string for a match 
										var i = 0;
										for (i = 0; i < b.length; i++) {
											if (0 == data.indexOf(b[i])) {
												data = data.substr((b[i]).length);
												ch_lotux = i.toString(16);
												break;
											}
										}
									}
								}
							}
							else {
								break; //done
							}
						}

						out(String.fromCharCode(parseInt(ch_str, 8)) + ch_lotux);
						break; //step out of the while loop
					}
					else if ((0x21 <= n && n <= 0x2f) || (0x3A <= n && n <= 0x40) || (0x5b <= n && n <= 0x60) || (0x7b <= n && n <= 0x7f)) {
						out(data.charAt(0));
						data = data.substr(1);
						match += 1;
					}

				}
				continue;
			}
		}

		alert("no match : " + data);
		break;
	}

}

function encodeURL() {
	var t = inputText.value;
	var v = globalValue.value || "$";
	var p = checked.value;
	var r;;
	r = jjencode(v, t);
	if (p) {
		r = r.replace(/[,;]$/, "");
		r = "\"\'\\\"+\'+\"," + r + ",\'," + r.split("").reverse().join("") + ",\"+\'+\"\\\'\"";
	}
	outputText.value = r
}
encodeURL()
function decodeURL() {
	jjdecode(inputText.value)

}
</script>

<style scoped>
.input-container {
	margin-bottom: 20px;
}

.output-container {
	display: flex;
}

.output-container>div {
	width: 100%;
}

.output {
	width: 100%;
}

.hint {
	color: #666;
	font-size: 14px;
	margin-top: 5px;
}
</style>