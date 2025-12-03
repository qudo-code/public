
# Encryptor
Class for handling encrypting/decrypting strings.
```ts
// Encryption utils
import crypto from 'crypto';

export const hash = (secret: string) => crypto.createHash('sha512').update(secret).digest('hex');

// Encryption class
/**
 * A class that provides encryption and decryption functionality using crypto.
 * @class
 * @example
 * // Initialize the encryptor
 * const encryptor = new Encryptoor({
 *   secretIv: 'your-iv-secret',
 *   secretKey: 'your-key-secret'
 * });
 *
 * // Encrypt some data
 * const encrypted = encryptor.encrypt('sensitive data');
 * console.log(encrypted); // Output: encrypted base64 string
 *
 * // Decrypt the data
 * const decrypted = encryptor.decrypt(encrypted);
 * console.log(decrypted); // Output: 'sensitive data'
 */
export class Encryptoor {
	/** Initialization vector used in the encryption process */
	iv: string;
	/** Encryption key */
	key: string;
	/** Encryption method/algorithm to be used */
	method: string;

	/**
	 * Creates an instance of Encryptoor.
	 * @param {Object} input - The input configuration object
	 * @param {string} input.secretIv - The secret value used to generate the initialization vector
	 * @param {string} input.secretKey - The secret value used to generate the encryption key
	 * @param {string} [input.method='aes-256-cbc'] - The encryption method/algorithm (defaults to 'aes-256-cbc')
	 */
	constructor(input: { secretIv: string; secretKey: string; method?: string }) {
		this.iv = hash(input.secretIv).substring(0, 16);
		this.key = hash(input.secretKey).substring(0, 32);
		this.method = input.method || 'aes-256-cbc';
	}

	/**
	 * Encrypts the provided data string.
	 * @param {string} data - The data to encrypt
	 * @returns {string} The encrypted data as a base64 string
	 * @throws {Error} If no data is provided
	 */
	encrypt(data: string) {
		if (!data) {
			throw new Error('no data to encrypt');
		}

		const cipher = crypto.createCipheriv(this.method, this.key, this.iv);
		// Encrypts data and converts to hex and base64
		return Buffer.from(cipher.update(data, 'utf8', 'hex') + cipher.final('hex')).toString('base64');
	}

	/**
	 * Decrypts the provided encrypted string.
	 * @param {string} encrypted - The encrypted data as a base64 string
	 * @returns {string} The decrypted data as a UTF-8 string
	 * @throws {Error} If no encrypted data is provided
	 */
	decrypt(encrypted: string) {
		if (!encrypted) {
			throw new Error('no data to decrypt');
		}

		const buff = Buffer.from(encrypted, 'base64');
		const decipher = crypto.createDecipheriv(this.method, this.key, this.iv);
		// Decrypts data and converts to utf8
		return decipher.update(buff.toString('utf8'), 'hex', 'utf8') + decipher.final('utf8');
	}
}
```
