<?php
/**
 很多童鞋应该都有用过 WordPress 的缩略图功能，暂且不说那些形形色色的缩略图插件，Wordpress 2.9 版本之后就新增自带了缩略图的功能，但是不知道你有没有和我同样的感觉，Wordpress 自带的缩略图还是有很大的局限性，比如说只能自动裁剪一个正方形的图片，而不能裁剪成长方形的；又比如说，只能接受站内或者说多媒体库里的图片，对站外图片就无从下手了；再比如说图片所在文件夹不是同一个，不方便管理。总之，Wordpress 自带的缩略图功能并没有我们想象的那样强大，这时候我们需要第三方文件 timthumb.php 项目来帮助我们。

现在我的主题上用到的缩略图都是 timthumb.php 来实现的，具体实例可以看我的首页幻灯片轮播部分，查看相应的幻灯图片的源码可以看到，图片的地址路径是 ****timthumb.php?src=****&h=245&w=560&zc=1 这样的格式，这就是被 timthumb.php 生成的缩略图，很神奇吧。

说了半天，什么是 timthumb.php 呢？这是一个专门为 WordPress 而开发的缩略图应用的项目。有点类似于插件，但是又和 WordPress 插件不同，因为它不是被上传于 plugins 文件夹下，而是需要上传到你的主题文件夹中。你可以在这里了解和下载最新版本的 timthumb.php，一般默认配置也就可以了，如果想进一步优化可以根据需要修改 timthumb.php 里前30行的参数。

说到 WordPress，纯文字有点太对不起观众，所以最后上一段我自用的缩略图代码，结合了 timthumb.php 和 WordPress 自带的缩略图功能，支持站外链接图片，自动缓存图片。代码如下：

**/
 
function post_thumbnail( $width = 100,$height = 80 ){
    global $post;
    if( has_post_thumbnail() ){    //如果有缩略图，则显示缩略图
        $timthumb_src = wp_get_attachment_image_src(get_post_thumbnail_id($post->ID),'full');
        $post_timthumb = '<img src="'.get_bloginfo("template_url").'/timthumb.php?src='.$timthumb_src[0].'&amp;h='.$height.'&amp;w='.$width.'&amp;zc=1" alt="'.$post->post_title.'" class="thumb" />';
        echo $post_timthumb;
    } else {
        $post_timthumb = '';
        ob_start();
        ob_end_clean();
        $output = preg_match('/<img.+src=[\'"]([^\'"]+)[\'"].*>/i', $post->post_content, $index_matches);    //获取日志中第一张图片
        $first_img_src = $index_matches [1];    //获取该图片 src
        if( !empty($first_img_src) ){    //如果日志中有图片
            $path_parts = pathinfo($first_img_src);    //获取图片 src 信息
            $first_img_name = $path_parts["basename"];    //获取图片名
            $first_img_pic = get_bloginfo('wpurl'). '/cache/'.$first_img_name;    //文件所在地址
            $first_img_file = ABSPATH. 'cache/'.$first_img_name;    //保存地址
            $expired = 604800;    //过期时间
            if ( !is_file($first_img_file) || (time() - filemtime($first_img_file)) > $expired ){
                copy($first_img_src, $first_img_file);    //远程获取图片保存于本地
                $post_timthumb = '<img src="'.$first_img_src.'" alt="'.$post->post_title.'" class="thumb" />';    //保存时用原图显示
            }
            $post_timthumb = '<img src="'.get_bloginfo("template_url").'/timthumb.php?src='.$first_img_pic.'&amp;h='.$height.'&amp;w='.$width.'&amp;zc=1" alt="'.$post->post_title.'" class="thumb" />';
        } else {    //如果日志中没有图片，则显示默认
            $post_timthumb = '<img src="'.get_bloginfo("template_url").'/images/default_thumb.gif" alt="'.$post->post_title.'" class="thumb" />';
        }
        echo $post_timthumb;
    }
}

/**
可以把这部分函数写进 WordPress 主题的 functions.php 里，然后再用 <?php post_thumbnail( 100,80 ) ?> 这样调用，其中的 $width 和 $height 是必须的参数哟。以上代码不多作解释了，注释都写的蛮详细了。今天就到这里，事情太多，洗洗睡了~

**/
