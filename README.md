			(!message.reply_message.audio && !message.reply_message.video)
		)
			return await message.send('*Reply to audio or video*')
		const p = await message.reply_message.downloadAndSaveMediaMessage('find')
		const data = await audioCut(p, 0, 15)
		const current_data = new Date()
		const timestamp = current_data.getTime() / 1000
		const stringToSign = buildStringToSign(
			'POST',
			options.endpoint,
			options.access_key,
			options.data_type,
			options.signature_version,
			timestamp
		)

		const signature = sign(stringToSign, options.access_secret)

		const form = new FormData()
		form.append('sample', data)
		form.append('sample_bytes', data.length)
		form.append('access_key', options.access_key)
		form.append('data_type', options.data_type)
		form.append('signature_version', options.signature_version)
		form.append('signature', signature)
		form.append('timestamp', timestamp)

		const res = await fetch('http://' + options.host + options.endpoint, {
			method: 'POST',
			body: form,
		})
		const { status, metadata } = await res.json()
		if (status.msg != 'Success')
			return await message.send(status.msg, { quoted: message.data })
		const { album, release_date, artists, title } = metadata.music[0]
		await message.send(
			`*Title :* ${title}
*Album :* ${album.name || ''}
*Artists :* ${
				artists !== undefined ? artists.map((v) => v.name).join(', ') : ''
			}
*Release Date :* ${release_date}`,
			{ quoted: message.quoted }
		)
	}
