# discourse-sso-for-yii2-starter-kit
Modified usage for yii2-starter-kit with discourse sso



You can get all the source code from the link below from original yii2-discourse-sso extension by sammaye
https://github.com/Sammaye/yii2-discourse-sso





# Samples Codes for yii2-starter-kit:

## 1. In discourse site, need set the url of sso url to https://www.testsite.com/user/sign-in/login  which is your yii2 framwork's login page

## 2. In yii2-starter-kit site, need put all the extension files into yii2-starter-kit/vendor/sammaye/yii2-discourse-sso/

## 3. In yii2-starter-kit site, need add the codes below in the yii2-starter-kit/vendor/yii2soft/extensions.php, you can composer install this extension, but it requires mongodb which is not used at all, so we'd better manully install the extension

```
  'sammaye/yii2-discourse-sso' => 
  array (
    'name' => 'sammaye/yii2-discourse-sso',
    'version' => '2.0.0.0',
    'alias' => 
    array (
      '@sammaye/discourse' => $vendorDir . '/sammaye/yii2-discourse-sso',
    ),
  ),
```

## 4. In the controller which is used for sso authenticating, we can add following function, this is almost same as the one in original doc, but added one line to make "@discourse" working
```
Yii::setAlias('@discourse', 'https://discourse site address');
```

### Same codes as original doc
```
	public function actionDiscourseSso()
	{
		$request = Yii::$app->getRequest();
		$sso = Yii::$app->discourseSso;
		
 		$payload = $request->get('sso');
		$sig = $request->get('sig');

		if(!($sso->validate($payload, $sig))){
			// invaild, deny
			throw new ForbiddenHttpException('Bad SSO request');
		}
		
		$nonce = $sso->getNonce($payload);
		
		if(Yii::$app->getUser()->isGuest){
			// We add session variable to track it after we log the user in so we can redirect them back
			// This method works well with custom login methods like social networks
			Yii::$app->getSession()->set('sso', ['sso' => $payload, 'sig' => $sig]);
			return $this->redirect(['site/login']);
		}else{
			$user = Yii::$app->getuser()->getIdentity();
		}
		
		Yii::$app->getSession()->remove('sso');
		
		// We send over the data
		$userparams = [
			"nonce" => $nonce,
			"external_id" => (String)$user->id,
			"email" => $user->email,
			
			// Optional - feel free to delete these two
			"username" => $user->username,
			"name" => $user->username,
			
			//'avatar_url' => Url::to(['image/profile-image', 'id' => (String)$user->_id], 'http')
		];
		$q = $sso->buildLoginString($userparams);
		
		/// We redirect back
		Yii::setAlias('@discourse', 'https://discourse site address');
		header('Location: ' . Yii::getAlias('@discourse') . '/session/sso_login?' . $q);
	 
	}
```

## 5. In the login funtion, need add following codes, original codes used session, but my code not, be careful about the redirect address, should be your controller/action address of sso function in step 4

File Location: yii2-starter-kit/frontend/modules/user/controllers/SignInController.php

Function changed " public function actionLogin()"

```
        if ($model->load(Yii::$app->request->post()) && $model->login()) {
			//Yii::warning($sso = Yii::$app->request->get('sso'));
			
			
			if($sso = Yii::$app->request->get('sso')){
				//Yii::warning('sso');
				$sig = Yii::$app->request->get('sig');
				
				//Yii::warning($sso);
				//Yii::warning($sig);
				
				return $this->redirect([
					'/your controller name/discourse-sso', 
					'sso' => $sso, 
					'sig' => $sig
				]);
			}
	
            return $this->goBack();
        } else {
			$this->layout = 'base_dev'; //change layout
            return $this->render('login', [
                'model' => $model
            ]);
        }

```

## end of yii2-starter-kit with discourse sso
##





# yii2-discourse-sso

Discourse SSO for Yii2

Currently this extension only really deals with logging in.

Installing and configuring is quite easy.

Make sure to [follow this guide on Discourse](https://meta.discourse.org/t/official-single-sign-on-for-discourse/13045) and while setting up SSO, after getting the secret needed, 
configure your application like so:

        'discourseSso' => [
        	'class' => 'sammaye\discourse\Sso',
        	'secret' => "Some super secret, super awesome dupa secret key"
        ],

That is this extension installed. Now you just need to learn how to use it.

Your extension is accessible via `Yii::$app->discourseSso`. This is the location used throughout this readme.

The first step is to setup a action in your `SiteController.php` to actually do the logic of signing in someone:

    public function actionDiscourseSso()
    {
    	$request = Yii::$app->getRequest();
    	$sso = Yii::$app->discourseSso;
    	
    	$payload = $request->get('sso');
    	$sig = $request->get('sig');

    	if(!($sso->validate($payload, $sig))){
    		// invaild, deny
    		throw new ForbiddenHttpException('Bad SSO request');
    	}
    	
    	$nonce = $sso->getNonce($payload);
    	
    	if(Yii::$app->getUser()->isGuest){
    		// We add session variable to track it after we log the user in so we can redirect them back
    		// This method works well with custom login methods like social networks
    		Yii::$app->getSession()->set('sso', ['sso' => $payload, 'sig' => $sig]);
    		return $this->redirect(['site/login']);
    	}else{
    		$user = Yii::$app->getuser()->getIdentity();
    	}
    	
    	Yii::$app->getSession()->remove('sso');
    	
    	// We send over the data
    	$userparams = [
	    	"nonce" => $nonce,
	    	"external_id" => (String)$user->_id,
	    	"email" => $user->email,
	    	
	    	// Optional - feel free to delete these two
	    	"username" => $user->username,
	    	"name" => $user->username,
	    	
	    	//'avatar_url' => Url::to(['image/profile-image', 'id' => (String)$user->_id], 'http')
    	];
    	$q = $sso->buildLoginString($userparams);
    	
    	/// We redirect back
    	header('Location: ' . Yii::getAlias('@discourse') . '/session/sso_login?' . $q);
    }
    
and even though this goes 90% of the way it does not complete it. If you notice:

	Yii::$app->getSession()->set('sso', ['sso' => $payload, 'sig' => $sig]);
	
We load the `sig` and `payload` into session so we can tell if we came from single sign-on so we can redirect back. This means that in 
our login functions (whether it be `login()` or some social login) we need to change it to (as an example for `login`):

    public function actionLogin()
    {
        if (!\Yii::$app->user->isGuest) {
            return $this->goHome();
        }

        $model = new LoginForm();
        if ($model->load(Yii::$app->request->post()) && $model->login()) {
        	
        	if($sso = Yii::$app->getSession()->get('sso')){
        		return $this->redirect([
        			'discourse-sso', 
        			'sso' => $sso['sso'], 
        			'sig' => $sso['sig']
				]);
        	}
        	
        	if($model->getUser()->profile_filled){
            	return $this->goBack();
        	}else{
        		return $this->redirect(['user/about-me']);
        	}
        } else {
            return $this->render('login', [
                'model' => $model,
            ]);
        }
    }
    
Notice the `if($sso = Yii::$app->getSession()->get('sso')){`, that is where he magic happens, it detects our single sign-on and redirects back to our single sign-on function.

To customise this, just add:

        if($sso = Yii::$app->getSession()->get('sso')){
        	return $this->redirect([
        		'discourse-sso', 
        		'sso' => $sso['sso'], 
        		'sig' => $sso['sig']
			]);
        }
        
to the end of every authentication function you have.
