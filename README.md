/*
                                  ▐ ▄ ▄▄▄ ..▄▄ · ▄▄▄▄▄ ▄· ▄▌
                                 •█▌▐█▀▄.▀·▐█ ▀. •██  ▐█▪██▌
                                 ▐█▐▐▌▐▀▀▪▄▄▀▀▀█▄ ▐█.▪▐█▌▐█▪
                                 ██▐█▌▐█▄▄▌▐█▄▪▐█ ▐█▌· ▐█▀·.
                                 ▀▀ █▪ ▀▀▀  ▀▀▀▀  ▀▀▀   ▀ • 
	Team Sev7ns - NestyStore.
	Ultimate Script Clear 0.0 A - Clear private Messages Discord.
	SCRIPT de cleardm de mensagens privadas do Discord.
	https://www.youtube.com/NestyStore
*/


/* ▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬Configurações do SCRIPT▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬ */
// Seu token de usuário do DISCORD
var authToken = 'SEU TOKEN'

// ID(s) de usuário(a) da(s) pessoa(s) ao qual as mensagens serão ignoradas.
// Aceita até 100 ID's.
//	Ex:
//		var blockedAuthors = ['1111111111111111111','22222222222222222222222',...]
if (typeof(blockedAuthors) === 'undefined') {
	var blockedAuthors = ['ID DA DM']
}

// ID da ultima mensagem enviada pra pessoa. (O SCRIPT começa apagando pela ultima mensagem enviada)
// É OPCIIONAL! Você pode deixar o SCRIPT rodar no automático
if (typeof(beforeId) === 'undefined') {
	var beforeId = null
}





/* ▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬Início do SCRIPT▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬ */
// Não faça nehuma alteração a partir daqui, se não poderá causar em inutilização do SCRIPT.
clearMessages = function() {
	//Cria conexão com o discordapp.
	const channel = window.location.href.split('/').pop()
	const baseURL = `https://discordapp.com/api/channels/${channel}/messages`
	const headers = { Authorization: authToken }

	//Inicia variavéis internas.
	let clock = 0
	let interval = 500
	let messagesStore = []

	//Delay do SCRIPT.
	function delay(duration) {
		return new Promise((resolve, reject) => {
			setTimeout(resolve, duration)
		})
	}

	//Carrega as mesagens.
	function loadMessages() {
		let url = `${baseURL}?limit=100`
		if (beforeId) {
			url += `&before=${beforeId}`
		}
		return fetch(url, { headers })
	}

	//Tenta apagar as mensagens.
	function tryDeleteMessage(message) {
		if (blockedAuthors.indexOf(message.author.id) === -1 && message.content != null) {
			console.log(`Mensagem de ${message.author.username} (${message.content.substring(0, 30)}...) apagada.`)
			return fetch(`${baseURL}/${message.id}`, { headers, method: 'DELETE' })
			beforeId = message.id
		}
	}

	//Filtra as mensagens por ID de usuário.
	function filterMessages(message) {
		return blockedAuthors.indexOf(message.author.id) === -1
	}

	//Rotina de mensagens não deletadas.
	function onlyNotDeleted(message) {
		return message.deleted === false
	}

	//Função principal do SCRIPT.
	loadMessages()
		.then(resp => resp.json())
		.then(messages => {
			if (messages === null || messages.length === 0) {
				console.log(`Todas as mensagens foram apagadas!`)
				console.log(`Recarregue a página para desligar o SCRIPT.`)
				return
			}
			beforeId = messages[messages.length-1].id
			messages.forEach(message => { message.deleted = false })
			messagesStore = messagesStore.concat(messages.filter(filterMessages))
			return Promise.all(messagesStore.filter(onlyNotDeleted).map(message => {
				return delay(clock += interval)
					.then(() => tryDeleteMessage(message))
					.then(resp => {
						if (resp && resp.status === 204) {
							message.deleted = true
							return resp.text()
						}
					})
					.then(result => {
						if (result) {
							result = JSON.parse(result)
							if (result.code === 50003) {
								console.log(`Não foi possível apagar a mensagem de ${message.author.username}, pulando-a`)
								blockedAuthors.push(message.author.id)
								messagesStore = messagesStore.filter(filterMessages)
							}
						}
					})
			}))
		})
		.then(function() {
			if (messagesStore.length !== 0 && messagesStore.length < 100) {
				clearMessages()
			} else {
				ClearLoop();
			}
		})
}

//Função para repetir o SCRIPT infinitamente.
function ClearLoop(){
	clearMessages()
}

//Roda o SCRIPT pela primeira vez.
ClearLoop()
