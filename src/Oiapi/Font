<?php
namespace Oiapi\Font;

use Utils\CoroutineCurl as curl;
use Utils\Yargs;
use App\BaseApp;
use Controller\Attr\Rule;

class Font extends BaseApp {
	const url = 'https://oiapi.net/api/XiaoMiFonts';
	#[Rule('/^\/?font.*$/', name: '/font + 指令')]
	public function help() {
		$message = $this->api->message;
		$Yargs = Yargs::getInstance();
		$Yargs->Command('/font', '字体功能')
			->positional('type', [
				'alias' => 't',
				'required' => true,
				'type' => 'string',
				'desc' => '模式',
				'default' => 'search',
				'choices' => ['search', 'get'],
			])->option('name', [
				'alias' => 'n',
				'required' => true,
				'type' => 'string',
				'desc' => '搜索时的字体名字',
			])->option('id', [
				'alias' => 'i',
				'required' => true,
				'type' => 'int',
				'desc' => '搜索时的选择',
			])->option('help', [
				'alias' => 'h',
				'required' => false,
				'type' => 'boolean',
				'desc' => '查看帮助',
			]);
		try {
			$Command = $Yargs->parse($this->raw);
			print_r($Command);
			switch($Command['arguments']['type']) {
				case 'search':
					if($Command['options']['h']) break;
					$name = $Command['options']['n'] ?? null;
					if(is_null($name)) return $message->sendGroupMsg($this->group, '缺少参数：-n');
					return $this->search($name);
					break;
				case 'get':
					if($Command['options']['h']) break;
					$id = $Command['options']['i'] ?? null;
					if(is_null($id)) return $message->sendGroupMsg($this->group, '缺少参数：-i');
					return $this->get($id);
					break;
				
			}
			return $message->sendGroupMsg($this->group, trim($Yargs->getHelp()));
		} catch (\Exception $e) {
			return $message->sendGroupMsg($this->group, trim($Yargs->getHelp()));
		}
	}
	private function search(?string $name = null, int $page = 1, int $limit = 12) {
		$message = $this->api->message;
		$url = self::url;
		$post = json_encode([
			'name' => $name,
			'page' => $page,
			'limit' => $limit,
		], 320);
		$curl = new curl;
		$curl->addheaders('content-type: application/json');
		$curl->post($url, $post);
		if($res = $curl->object()) {
			if($res->code !== 1) return $message->sendGroupMsg($this->group, $res->message);
			$this->redis->set("{$this->qq}.searchFont", $post);
			$message->sendGroupMsg($this->group, '搜索完成，正在发送列表，请稍等…');
			$Node = [];
			foreach($res->data as $val) {
				array_push($Node, $message->structureNode(name: $this->nickname, uin: $this->qq, content: [
					$message->structureImage($val->cover),
					$message->structureText("{$val->author}：{$val->name}\n大小：{$val->size}")
				]));
			}
			return $message->sendGroupForwardMsg($this->group, $Node);
		}
		return $message->sendGroupMsg($this->group, '获取失败，网络错误');
	}
	private function get(int $id) {
		$message = $this->api->message;
		if($id < 1) return $message->sendGroupMsg($this->group, '参数错误，id不能小于1');
		if(!$this->redis->exists("{$this->qq}.searchFont")) return $message->sendGroupMsg($this->group, '请先搜索');
		$post = json_decode($this->redis->get("{$this->qq}.searchFont"));
		$post->n = $id;
		$curl = new curl;
		$curl->addheaders('content-type: application/json');
		$curl->post(self::url, json_encode($post, 320));
		if($res = $curl->object()) {
			if($res->code !== 1) return $message->sendGroupMsg($this->group, $res->message);
			$Node = [];
			$val = $res->data;
			array_push($Node, $message->structureNode(name: $this->nickname, uin: $this->qq, content: [
				$message->structureImage($val->cover),
				$message->structureText("{$val->author}：{$val->name}\n大小：{$val->size}"),
				$message->structureText("\n下载链接：{$val->download}"),
			]));
			return $message->sendGroupForwardMsg($this->group, $Node);
		}
		return $message->sendGroupMsg($this->group, '获取失败，网络错误');
	}
}