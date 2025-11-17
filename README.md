<?php
session_start();

// === Database Connection ===
$host = "localhost";
$user = "root";
$pass = "";
$db   = "emote_bot";

$conn = mysqli_connect($host, $user, $pass, $db);
if(!$conn){
    die("Database connection failed: ".mysqli_connect_error());
}

// === Handle Signup ===
if(isset($_POST['action']) && $_POST['action']=="signup"){
    $username = $_POST['username'];
    $password = $_POST['password'];
    
    $check = mysqli_query($conn,"SELECT * FROM users WHERE username='$username'");
    if(mysqli_num_rows($check) > 0){
        echo "exist"; exit;
    }

    $hash = password_hash($password, PASSWORD_DEFAULT);
    $sql = "INSERT INTO users(username,password) VALUES('$username','$hash')";
    if(mysqli_query($conn,$sql)){
        echo "success"; exit;
    }else{
        echo "error"; exit;
    }
}

// === Handle Login ===
if(isset($_POST['action']) && $_POST['action']=="login"){
    $username = $_POST['username'];
    $password = $_POST['password'];

    $q = mysqli_query($conn,"SELECT * FROM users WHERE username='$username'");
    if(mysqli_num_rows($q) == 0){
        echo "invalid"; exit;
    }

    $data = mysqli_fetch_assoc($q);
    if(password_verify($password, $data['password'])){
        $_SESSION['user'] = $username;
        echo "success"; exit;
    }else{
        echo "invalid"; exit;
    }
}

// === Handle Logout ===
if(isset($_GET['logout'])){
    session_destroy();
    header("Location:index.php");
    exit;
}

// === If logged in, show dashboard ===
if(isset($_SESSION['user'])): ?>
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Dashboard - Free Fire Emote Bot</title>
<style>
body{background:#111;color:white;font-family:sans-serif;text-align:center;padding:50px;}
h1{color:#FF6B00;}
a{color:#FFD700;text-decoration:none;font-weight:bold;}
button{padding:12px 24px;background:#FF6B00;border:none;border-radius:8px;color:white;cursor:pointer;margin-top:20px;}
button:hover{background:#FF3D00;}
</style>
</head>
<body>
<h1>Welcome, <?php echo $_SESSION['user']; ?>!</h1>
<p>You are logged in to <strong>Free Fire Emote Bot Dashboard</strong></p>
<a href="?logout=1"><button>Logout</button></a>
</body>
</html>
<?php exit; endif; ?>

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Free Fire Emote Bot</title>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
<style>
:root {
    --primary:#FF6B00;
    --secondary:#FF3D00;
    --accent:#FFD700;
    --dark:#0A0A0A;
    --darker:#050505;
    --dark-card:#1A1A1A;
    --light:#FFFFFF;
    --gray:#8C8C8C;
    --radius:12px;
    --shadow-lg:0 16px 48px rgba(0,0,0,0.3);
    --transition:all 0.3s cubic-bezier(0.4,0,0.2,1);
}

*{margin:0;padding:0;box-sizing:border-box;font-family:'Inter',sans-serif;}
body{background: linear-gradient(135deg,var(--darker)0%,var(--dark)50%,#1A0F0F 100%);min-height:100vh;display:flex;align-items:center;justify-content:center;padding:20px;position:relative;overflow-x:hidden;}

.background{position:fixed;top:0;left:0;width:100%;height:100%;z-index:0;overflow:hidden;}
.particle{position:absolute;background:radial-gradient(circle,var(--primary)0%,transparent 70%);border-radius:50%;opacity:0.1;animation:float 20s infinite linear;}
.particle:nth-child(1){width:400px;height:400px;top:-200px;left:-200px;animation-duration:25s;}
.particle:nth-child(2){width:300px;height:300px;bottom:-150px;right:-150px;animation-duration:20s;animation-direction:reverse;}
.particle:nth-child(3){width:200px;height:200px;top:20%;right:15%;animation-duration:15s;}
@keyframes float{0%{transform:rotate(0deg) translateX(0) translateY(0);}25%{transform:rotate(90deg) translateX(20px) translateY(-20px);}50%{transform:rotate(180deg) translateX(0) translateY(-40px);}75%{transform:rotate(270deg) translateX(-20px) translateY(-20px);}100%{transform:rotate(360deg) translateX(0) translateY(0);}}

.container{display:grid;grid-template-columns:1fr 1fr;max-width:1000px;width:100%;background:rgba(26,26,26,0.85);backdrop-filter:blur(20px);border-radius:var(--radius);overflow:hidden;box-shadow:var(--shadow-lg);border:1px solid rgba(255,107,0,0.2);position:relative;z-index:1;animation:slideUp 0.8s;}
@keyframes slideUp{from{opacity:0;transform:translateY(40px) scale(0.95);}to{opacity:1;transform:translateY(0) scale(1);}}

.hero-section{background:linear-gradient(135deg,rgba(255,107,0,0.1)0%,rgba(255,61,0,0.05)100%);padding:60px 50px;display:flex;flex-direction:column;justify-content:center;align-items:center;text-align:center;position:relative;overflow:hidden;}
.hero-section h1{font-size:36px;font-weight:700;color:var(--light);margin-bottom:16px;}
.hero-section p{font-size:16px;color:var(--gray);line-height:1.6;margin-bottom:40px;max-width:400px;}
.logo-icon{font-size:80px;color:var(--accent);text-shadow:0 0 40px rgba(255,215,0,0.5);margin-bottom:15px;animation:glow 2s ease-in-out infinite alternate;}
@keyframes glow{from{filter:drop-shadow(0 0 10px rgba(255,215,0,0.3));transform:scale(1);}to{filter:drop-shadow(0 0 20px rgba(255,215,0,0.6));transform:scale(1.05);}}
.logo-text{font-size:32px;font-weight:800;background:linear-gradient(135deg,var(--accent)0%,var(--primary)100%);-webkit-background-clip:text;-webkit-text-fill-color:transparent;}

.form-section{padding:60px 50px;display:flex;flex-direction:column;justify-content:center;background:var(--dark-card);}
.form-group{margin-bottom:24px;}
.form-input{width:100%;padding:16px 20px;border:2px solid var(--gray);border-radius:var(--radius);font-size:16px;background:rgba(255,255,255,0.05);color:var(--light);}
.form-input:focus{outline:none;border-color:var(--primary);box-shadow:0 0 0 4px rgba(255,107,0,0.1);}
.btn{background:linear-gradient(135deg,var(--primary),var(--secondary));color:var(--light);border:none;padding:18px 32px;border-radius:var(--radius);font-weight:700;cursor:pointer;transition:var(--transition);width:100%;display:flex;align-items:center;justify-content:center;gap:12px;position:relative;overflow:hidden;}
.btn:hover{transform:translateY(-2px);box-shadow:0 12px 24px rgba(255,107,0,0.3);}
.switch{text-align:center;margin-top:12px;color:var(--accent);cursor:pointer;}
.alert{color:#FF4757;text-align:center;margin-bottom:12px;font-weight:600;}

@media(max-width:968px){.container{grid-template-columns:1fr;max-width:500px;}.hero-section,.form-section{padding:40px 30px;}}
@media(max-width:480px){.hero-section,.form-section{padding:30px 20px;}.logo-icon{font-size:60px;}.logo-text{font-size:28px;}.hero-section h1{font-size:28px;}}
</style>
</head>
<body>

<div class="background">
    <div class="particle"></div>
    <div class="particle"></div>
    <div class="particle"></div>
</div>

<div class="container">
    <div class="hero-section">
        <div class="logo-icon"><i class="fas fa-fire-flame-curved"></i></div>
        <div class="logo-text">FREE FIRE</div>
        <h1 id="heroTitle">Welcome Back, Soldier!</h1>
        <p id="heroDesc">Sign in to access your exclusive Free Fire emotes and continue dominating the battlefield with style.</p>
    </div>
    
    <div class="form-section">
        <div class="alert" id="alert"></div>
        <form id="authForm">
            <div class="form-group"><input type="text" name="username" placeholder="Telegram Username" required></div>
            <div class="form-group"><input type="password" name="password" placeholder="Password" required></div>
            <button type="submit" class="btn" id="submitBtn">Sign In</button>
        </form>
        <div class="switch" id="switchForm">Don't have an account? Sign Up</div>
    </div>
</div>

<script>
let form=document.getElementById('authForm');
let alertBox=document.getElementById('alert');
let submitBtn=document.getElementById('submitBtn');
let switchForm=document.getElementById('switchForm');
let heroTitle=document.getElementById('heroTitle');
let heroDesc=document.getElementById('heroDesc');
let mode="login";

switchForm.addEventListener('click',()=>{
    if(mode=="login"){
        mode="signup";
        submitBtn.innerText="Create Account";
        switchForm.innerText="Already have an account? Sign In";
        heroTitle.innerText="Join the Elite Squad";
        heroDesc.innerText="Create your account and unlock exclusive Free Fire emotes. Dominate the battlefield with style and express yourself like never before.";
    }else{
        mode="login";
        submitBtn.innerText="Sign In";
        switchForm.innerText="Don't have an account? Sign Up";
        heroTitle.innerText="Welcome Back, Soldier!";
        heroDesc.innerText="Sign in to access your exclusive Free Fire emotes and continue dominating the battlefield with style.";
    }
    alertBox.innerText="";
});

form.addEventListener('submit',function(e){
    e.preventDefault();
    alertBox.innerText="";
    let data=new FormData(form);
    data.append('action',mode);
    submitBtn.disabled=true;
    submitBtn.innerText=mode=="login"?"Signing In...":"Creating Account...";

    fetch("",{method:"POST",body:data})
    .then(res=>res.text())
    .then(res=>{
        submitBtn.disabled=false;
        submitBtn.innerText=mode=="login"?"Sign In":"Create Account";
        if(res=="success"){
            if(mode=="login"){window.location.href="";}
            else{
                alertBox.style.color="#2ED573";
                alertBox.innerText="Account created! You can login now.";
                mode="login";
                submitBtn.innerText="Sign In";
                switchForm.innerText="Don't have an account? Sign Up";
                heroTitle.innerText="Welcome Back, Soldier!";
                heroDesc.innerText="Sign in to access your exclusive Free Fire emotes and continue dominating the battlefield with style.";
            }
        }else if(res=="exist"){alertBox.style.color="#FF4757";alertBox.innerText="Username already exists!";}
        else if(res=="invalid"){alertBox.style.color="#FF4757";alertBox.innerText="Invalid username or password!";}
        else{alertBox.style.color="#FF4757";alertBox.innerText="Something went wrong!";}
    });
});
</script>

</body>
</html>
