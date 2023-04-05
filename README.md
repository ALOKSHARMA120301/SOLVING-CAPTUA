# SOLVING-CAPTUA
You need to write a Node.js Script which takes an image public URL as an input and returns the solved Captcha
const axios = require('axios');
const FormData = require('form-data');

const API_KEY = 'your_2captcha_api_key';
const CAPTCHA_URL = 'https://example.com/captcha.jpg';

async function solveCaptcha() {
  // Step 1: Get Captcha image as binary data
  const response = await axios.get(CAPTCHA_URL, {
    responseType: 'arraybuffer'
  });
  const captchaImage = response.data;

  // Step 2: Upload Captcha image to 2Captcha API
  const formData = new FormData();
  formData.append('key', API_KEY);
  formData.append('method', 'post');
  formData.append('file', captchaImage, {
    filename: 'captcha.jpg',
    contentType: 'image/jpeg'
  });
  const uploadResponse = await axios.post('https://2captcha.com/in.php', formData, {
    headers: formData.getHeaders()
  });

  // Step 3: Wait for Captcha to be solved
  const captchaId = uploadResponse.data.split('|')[1];
  const resultResponse = await axios.get(`https://2captcha.com/res.php?key=${API_KEY}&action=get&id=${captchaId}&json=1`);
  while (resultResponse.data.status === 0) {
    await new Promise(resolve => setTimeout(resolve, 5000)); // Wait 5 seconds before checking again
    resultResponse = await axios.get(`https://2captcha.com/res.php?key=${API_KEY}&action=get&id=${captchaId}&json=1`);
  }

  // Step 4: Get solved Captcha from API response
  const solvedCaptcha = resultResponse.data.request;

  return solvedCaptcha;
}

solveCaptcha()
  .then(captcha => console.log(`Solved Captcha: ${captcha}`))
  .catch(error => console.error(error));
