# OTP
```ts
// 2FA utils
import QRCode from 'qrcode';
import speakeasy from 'speakeasy';

export const generateSecret = async () => {
	const secret = speakeasy.generateSecret();

	return {
		otpUrl: secret.otpauth_url,
		secret: secret.base32,
	};
};

export const secretToOtpAuthUrl = (secret: string, username: string) => {
	const url = new URL(`otpauth://totp/${secret}?secret=${secret}&issuer=My%20Profiles - ${username}`);

	return url.toString();
};

export const urlToQrCode = async (url: string) => {
	const qrCodeUrl = await QRCode.toDataURL(url.toString());

	return qrCodeUrl;
};

// Verify a token from users authenticator app or from generateToken().
export const verifyToken = async (secret: string, token: string) => {
	const verified = speakeasy.totp.verify({
		secret,
		encoding: 'base32',
		token,
	});

	return verified;
};

// Generate a token from their secret. Used for SMS or Email 2FA.
export const generateToken = async (secret: string) => {
	const token = speakeasy.totp({
		secret,
		encoding: 'base32',
	});

	return token;
};
```
