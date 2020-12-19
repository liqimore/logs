---
layout: post
title: "jekyll reconfigured test"
description: "jekyll reconfigured test."
date: 2020-12-19
tags: [test]
categories: [test]
comments: true
---

## Nginx反向代理的设置方法
nginx反向代理有很多的不同方面的用处, 这里主要讲如何设置docker的反向代理, 把本地的docker环境映射到域名上去.  
### 系统环境
docker + nginx + 公网IP + 域名  
### 操作方法
1. 首先在启动docker时, 把docker内部端口使用`-p 外部端口:内部端口`映射到服务器的网络环境中, 不用给当前的docker容器创建docker网络环境.  
2. 然后编写nginx配置文件, 我是用oneinstack可以直接创建nginx的conf配置并且自动生成免费的SSL证书, 操作简单.
3. 删掉oninstack在nginx配置文件中生成的对图片和其他资源文件处理的location配置, 加入如下的反向代理语句:   
```
location / {
        proxy_set_header  Host  $http_host;
        proxy_set_header  X-Real-IP  $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass  http://127.0.0.1:6001; #自行修改为你需要的配置
    }
```
上面的配置可以把6001端口的服务代理到其他端口,我在使用过程中是代理到了80端口, 之后使用域名进行解析.  

## 图片cdn测试
### 小图
![房间](/assets/images/202012/room.jpg)
### 大图
![sky](/assets/images/202012/sky.jpg)

## 代码高亮
```php
<?php

namespace App\Http\Controllers;

use App\Models\Page;
use App\Models\Paste;
use App\Models\Syntax;
use Illuminate\Http\Request;
use Mail;
use Validator;

class PageController extends Controller
{

    public function show($slug)
    {
        $page = Page::where('slug', $slug)->where('active', 1)->firstOrfail();

        $old_page = \DB::table('pages')->where('slug',$slug)->where('active',1)->first();

        $result = json_decode($old_page->content);

        if (json_last_error() !== 0) {
            
            $page->title = $old_page->title;
            $page->content = $old_page->content;
            $page->save();
        }

        $description = trim(preg_replace('/\s+/', ' ', strip_tags($page->content)));

        $page->description = str_limit($description, 200, '');

        return view('front.page.show', compact('page'))->with('page_title', $page->title);
    }

    public function contact()
    {
        return view('front.page.contact')->with('page_title', __('Contact Us'));
    }

    public function contactPost(Request $request)
    {
        $captcha = '';
        if (config('settings.captcha') == 1) {
            // if (config('settings.captcha_type') == 1) {
            //     $captcha = 'required|captcha';
            // } else {
            //     $captcha = 'required|custom_captcha';
            // }
            $captcha = 'required|captcha';

        }

        $validator = Validator::make($request->all(), [
            'name' => 'required|eco_alpha_spaces|min:2|max:100',
            'email' => 'required|email|max:100',
            'message' => 'required|string|min:10|max:5000',
            'g-recaptcha-response' => $captcha,
        ]);
        if ($validator->fails()) {
            return redirect()->back()
                ->withErrors($validator)
                ->withInput();
        } else {

            try {
                Mail::send('emails.contact', ['request' => $request], function ($m) {
                    $m->to(config('settings.site_email'))->subject(config('settings.site_name') . ' - ' . __('Contact Message'));
                });
            } catch (\Exception $e) {
                \Log::info($e->getMessage());
                return redirect('contact')->with('warning', __('Your message was not sent due to invalid mail configuration'));
            }

            return redirect('contact')->with('success', __('Your message successfully sent'));
        }

    }

    public function sitemaps()
    {
        $first_product = Paste::orderBy('created_at')->firstOrfail(['created_at']);

        $last_product = Paste::orderBy('created_at','DESC')->firstOrfail(['created_at']);

        $start_date = $first_product->created_at->format('Y-m-d');
        $end_date = $last_product->created_at->format('Y-m-d');

        return response()->view('front.page.sitemaps',compact('start_date','end_date'))->header('Content-Type', 'text/xml');
    }


    public function sitemapMain()
    {
        $pages = Page::where('active', 1)->get(['slug']);

        $syntaxes = Syntax::where('active', 1)->get(['slug']);
        return response()->view('front.page.sitemap_main', compact('pages', 'syntaxes'))->header('Content-Type', 'text/xml');
    }

    public function sitemap($date)
    {
        $pastes = Paste::where('status', 1)->where(function ($query) {
            $query->where('expire_time', '>', \Carbon\Carbon::now())->orWhereNull('expire_time');
        })->where(function ($query) {
            $query->orWhereNull('user_id');
            $query->orWhereHas('user', function ($user) {
                $user->whereIn('status', [0, 1]);
            });
        })->whereDate('created_at',$date)
        ->orderBy('created_at', 'DESC')->get(['id', 'slug']);

        $users = \App\User::where('status',1)->whereDate('created_at',$date)->orderBy('created_at','DESC')->get(['id','name','avatar','role','status','about','fb','tw','gp']);

        return response()->view('front.page.sitemap',compact('pastes','users'))->header('Content-Type', 'text/xml');
    }

    public function redirect(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'url' => 'required|url',
        ]);
        if ($validator->fails()) {
            return redirect()->back()
                ->withErrors($validator);
        } 
        $url = $request->url;
        return view('front.page.redirect',compact('url'))->With('page_title',__('Leaving').' '.config('settings.site_name'));
    }
}
```



## 参考
http://wiki.jikexueyuan.com/project/nginx/load-balancing-module.html  
http://tengine.taobao.org/book/chapter_05.html#upstream  
https://www.cnblogs.com/youzhibing/p/7327342.html  
