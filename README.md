# kvs-automatic-extraction
 中文TAG自动提取
 中文标题转拼音SEO链接

## Demo
测试标题  
	你好世界
自动提取Tag
	你好,世界
自动生成SEO链接
	ni-hao-shi-jie


## Install

```shell

export COMPOSER_ALLOW_SUPERUSER=1
composer require "overtrue/pinyin:~4.0"
composer require fukuball/jieba-php:dev-master
composer require yurunsoft/chinese-util

```


## Update Files


```shell

# admin/plugins/tags_autogeneration/tags_autogeneration.php

## In Top
require_once dirname(dirname(dirname(dirname(__FILE__))))."/vendor/autoload.php";
use Overtrue\Pinyin\Pinyin;
$pinyin = new Pinyin('\\Overtrue\\Pinyin\\MemoryFileDictLoader');
use Fukuball\Jieba\Jieba;
use Fukuball\Jieba\Finalseg;
use Fukuball\Jieba\JiebaAnalyse;
Jieba::init(array('mode'=>'default','dict'=>'small'));
JiebaAnalyse::init();
Finalseg::init();

## Update Function

function tags_autogenerationGenerate($title,$description,$tags_all,$lenient,$lenient_list)
{
	$tags_found=array();

	$punkt=array(".",",",":",";","-","+","=","'","\"","(",")","`");
	foreach ($tags_all as $k=>$v)
	{
		if (strpos($k,' ')!==false || strpos($k,'-')!==false)
		{
			if (strpos($title,$k)!==false || strpos($description,$k)!==false)
			{
				$tags_found[$k]=$v;
			} elseif ($lenient==1 || ($lenient==2 && is_array($lenient_list) && $lenient_list[$k]>0))
			{
				$lenient_words=explode(' ',$k);
				$all_words_match=true;
				foreach ($lenient_words as $lenient_word)
				{
					if (strpos($title,$lenient_word)===false && strpos($description,$lenient_word)===false)
					{
						$all_words_match=false;
						break;
					}
				}
				if ($all_words_match)
				{
					$tags_found[$k]=$v;
				}
			}
		}
	}
	$title=str_replace($punkt," ",$title);
	$description=str_replace($punkt," ",$description);

    // Process Tags
    $clear_title = $title;
    $char = "。、！？：；﹑•＂…‘’“”〝〞∕¦‖—　〈〉﹞﹝「」‹›〖〗】【»«』『〕〔》《﹐¸﹕︰﹔！¡？¿﹖﹌﹏﹋＇´ˊˋ―﹫︳︴¯＿￣﹢﹦﹤‐­˜﹟﹩﹠﹪﹡﹨﹍﹉﹎﹊ˇ︵︶︷︸︹︿﹀︺︽︾ˉ﹁﹂﹃﹄︻︼（）";
    $pattern = [
        "/[[:punct:]]/i", //英文标点符号
        '/['.$char.']/u', //中文标点符号
        '/[ ]{2,}/', //
    ];
    $str_title = preg_replace($pattern, ' ', $clear_title);
    $extract_tags = JiebaAnalyse::extractTags($str_title, 30);
    $temp = implode(" ",array_keys($extract_tags));
////    $temp = explode(",", $lines);
//    $temp = array_keys($extract_tags);
//    $temp = array_map("trim", $temp);
//    $temp = array_unique($temp);
    $temp=explode(" ",$temp);
//	$temp=array_merge(explode(" ",$title),explode(" ",$description));
	foreach ($temp as $candidate)
	{
		$candidate=trim($candidate);
		if (strlen($candidate)<1 || $tags_found[$candidate]>0)
		{
			continue;
		}
		if ($tags_all[$candidate]>0)
		{
			$tags_found[$candidate]=$tags_all[$candidate];
		}
	}
	return $tags_found;
}


# Update admin/inlcude/function_base.php 1272 Line

	global $config;

	$str=trim($str);
	if ($str=='') { return $str; }

	require_once $config[project_path]."/vendor/autoload.php";
	$pinyin = new \Overtrue\Pinyin\Pinyin('\\Overtrue\\Pinyin\\MemoryFileDictLoader');
	$dir = $pinyin->permalink($str);
	if (strlen($dir) > 250) { //字符串超限 , Over string num limit
		$dir = $pinyin->abbr($str,PINYIN_KEEP_NUMBER);
	}
	$temp_dir=$dir;
	for ($i=2;$i<999999;$i++){
		if (mr2number(sql_pr("select count(*) from $config[tables_prefix]videos where dir=?",$temp_dir))==0)
	{
	    $dir=$temp_dir;break;
	}
	$temp_dir=$dir.$i;
	}
	$dir = strtolower($temp_dir);
	return $dir;

	$options=get_options(array('DIRECTORIES_TRANSLIT','DIRECTORIES_TRANSLIT_RULES','DIRECTORIES_MAX_LENGTH'));


```


