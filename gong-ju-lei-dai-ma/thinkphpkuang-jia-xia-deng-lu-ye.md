```
<?php

namespace Admin\Controller;

use Common\ORG\phpMethod;
use Think\Controller;
use Think\Log;
use Common\ORG\Image;
class LoginController extends Controller
{
    public function index()
    {
        $this->display();
    }
    function login()
    {
        $message = '';
        $content = '';
        $callback = I('callback');
        $success = true;
        $password = I('password', '', 'md5');
        $userid = I('UserName', '', 'strip_tags');
        if (empty ($userid)) {
            phpMethod::log('帐号不能为空！');
        } elseif (empty ($password)) {
            phpMethod::log('密码不能为空！');
        }
       if ($_SESSION ['verify'] != md5(strtolower($_GET['verify']))) {
            phpMethod::jsonResult(null, '验证码错误！', $callback);
        }
        $map = array(
            'username' => $userid,
            'password' => $password,
        );
        $authInfo = M('admin')->where($map)->find();
        if (!empty($authInfo)) {
            $_SESSION [C('ADMIN_AUTH_KEY')] = $authInfo ['username'];
            $content = "登陆成功！";

            $data = array(
                'last_login_time' => time(),
                'last_login_ip' => phpMethod::getIP(),
            );
            M("admin")->where("userid=" . $authInfo['userid'])->data($data)->save();
        } else {
            $message = "用户名密码错误!";
            $success = false;
            phpMethod::log("用户名密码错误" . M()->getLastSql());
        }
        phpMethod::jsonResult($content, $message, $callback, $success);
    }
    Public function verify()
    {
        ob_clean();
        import('ORG.Util.Image');
        // import('ORG.Util.String');
        Image::buildImageVerify(4, 1);
    }
    // 用户登出
    public function logout()
    {
        session(null);
        session_destroy();
        unset ($_SESSION);
        session(C('ADMIN_AUTH_KEY'), null);
        phpMethod::jsonResult(1);
    }
}

?>
```



