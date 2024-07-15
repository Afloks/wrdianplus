const prizes = [
    'خصم 10%',
    'خصم 20%',
    'خصم 30%',
    'خصم 40%',
    'خصم 50%',
    'خصم 60%',
    'خصم 70%',
    'خصم 80%',
    'خصم 90%',
    'حظ أوفر المرة القادمة'
];

let attempts = 0;
const maxAttempts = 5;

function login() {
    const username = document.getElementById('username').value;
    const phone = document.getElementById('phone').value;
    const email = document.getElementById('email').value;
    
    if (username && phone && email) {
        const userData = {
            username,
            phone,
            email,
            points: 0 // يمكن تحديث النقاط لاحقاً
        };

        localStorage.setItem(username, JSON.stringify(userData));

        showUserInfo(userData);
        document.getElementById('login-container').style.display = 'none';
        document.getElementById('scratch-container').style.display = 'block';
        resetCard();
    } else {
        showToast('يرجى ملء جميع الحقول.');
    }
}

function showUserInfo(userData) {
    const userDetails = document.getElementById('user-details');
    userDetails.innerHTML = `
        <strong>اسم المستخدم:</strong> ${userData.username}<br>
        <strong>رقم الهاتف:</strong> ${userData.phone}<br>
        <strong>البريد الإلكتروني:</strong> ${userData.email}<br>
        <strong>النقاط:</strong> ${userData.points}
    `;
    document.getElementById('user-info').style.display = 'block';
}

function logout() {
    const username = document.getElementById('username').value;
    localStorage.removeItem(username);
    document.getElementById('user-info').style.display = 'none';
    document.getElementById('login-container').style.display = 'block';
    document.getElementById('scratch-container').style.display = 'none';
    document.getElementById('username').value = '';
    document.getElementById('phone').value = '';
    document.getElementById('email').value = '';
}

let isDrawing = false;
let lastX = 0;
let lastY = 0;
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const prize = document.getElementById('prize');

canvas.addEventListener('mousedown', (e) => {
    isDrawing = true;
    [lastX, lastY] = [e.offsetX, e.offsetY];
});

canvas.addEventListener('mousemove', draw);
canvas.addEventListener('mouseup', () => isDrawing = false);
canvas.addEventListener('mouseout', () => isDrawing = false);

function draw(e) {
    if (!isDrawing) return;
    ctx.globalCompositeOperation = 'destination-out';
    ctx.lineWidth = 20;
    ctx.lineJoin = 'round';
    ctx.lineCap = 'round';
    ctx.strokeStyle = 'rgba(0,0,0,1)';

    ctx.beginPath();
    ctx.moveTo(lastX, lastY);
    ctx.lineTo(e.offsetX, e.offsetY);
    ctx.stroke();
    [lastX, lastY] = [e.offsetX, e.offsetY];
}

function resetCard() {
    const username = document.getElementById('username').value;
    let userData;
    try {
        userData = JSON.parse(localStorage.getItem(username));
        if (!userData) {
            userData = { lastLogin: null, attempts: 0 };
        }
    } catch (error) {
        console.error('Error parsing user data:', error);
        userData = { lastLogin: null, attempts: 0 };
    }

    if (userData.attempts >= maxAttempts) {
        showToast('لقد تجاوزت الحد الأقصى للمحاولات لهذا الأسبوع.');
        return;
    }

    userData.attempts += 1;
    localStorage.setItem(username, JSON.stringify(userData));
    
    ctx.globalCompositeOperation = 'source-over';
    ctx.fillStyle = '#ccc';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    const selectedPrize = prizes[Math.floor(Math.random() * prizes.length)];
    prize.textContent = selectedPrize;
    prize.style.display = 'none';
    
    setTimeout(() => {
        prize.style.display = 'block';
        if (selectedPrize !== 'حظ أوفر المرة القادمة') {
            confetti();
            playSound('sounds/win.mp3');
        }
    }, 500);
}

function confetti() {
    canvasConfetti({
        particleCount: 100,
        spread: 70,
        origin: { y: 0.6 }
    });
}

function playSound(url) {
    const audio = new Audio(url);
    audio.play().catch(error => console.error('Error playing sound:', error));
}

function showToast(message) {
    const toast = document.getElementById('toast');
    toast.textContent = message;
    toast.style.display = 'block';
    setTimeout(() => {
        toast.style.display = 'none';
    }, 3000);
}
