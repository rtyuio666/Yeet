const fs = require('fs');
const config = require('./config.json');
const discord  = require('discord.js');
const carbon = new discord.Client();
const prefix = config.prefix;

carbon.commands = new discord.Collection();
carbon.aliases = new discord.Collection();

const categories = ['Fun', 'Info', 'NSFW', 'Utility'];

for (let i = 0; i < categories.length; i++) {
	let path = `${categories[i]}`;
	fs.readdir(`./commands/${categories[i]}`, (err, files) => {
		console.log(`Category: ${categories[i]} loaded.`);
		if (err) console.error(err);
		let jsfile = files.filter(f => f.split(".").pop() === 'js');
		if (jsfile.length <= 0) {
			return console.error("[LOGS] No commands/categories found");
		}
		jsfile.forEach((f, i) => {
			let pull = require(`./commands/${path}/${f}`);
			carbon.commands.set(pull.config.name, pull);
			pull.config.aliases.forEach(alias => {
				carbon.aliases.set(alias, pull.config.name);
			});
		});
	});
}

carbon.on('ready', () => {
	console.log('ready');
	carbon.user.setActivity(`${config.prefix}help`, {
		type: "WATCHING"
	});
});

carbon.on('guildCreate', async guild => {
	guild.channels.first().createInvite().then(inv => {
		carbon.channels.get('547503764542849044').send(`Carbon has been added to a new server.\n\nInfo:
		\`\`\`
		Owner: ${guild.owner.user.tag}
		Server Name: ${guild.name}
		Member Count: ${guild.memberCount.toLocaleString()}
		Bots: ${guild.members.filter(m => m.user.bot).size}
		Invite Link: ${inv.url}\`\`\``);
	});
});

carbon.on('message', async message => {
	console.log(`${message.guild.name} ${message.guild.owner.user.tag} | ${message.author.tag}: ${message.content}`);
	if (!message.content.startsWith(prefix.toLowerCase() || prefix.toUpperCase())) return;
	if (!message.guild) return;
	if (message.author.bot) return;

	const args = message.content.slice(prefix.length).trim().split(/ +/g);
	const command = args.shift().toLowerCase();

	let commandfile = carbon.commands.get(command) || carbon.commands.get(carbon.aliases.get(command));
	if (commandfile) commandfile.run(carbon, message, args);
});

carbon.login(config.token);
