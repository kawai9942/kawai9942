const { Client, GatewayIntentBits } = require('discord.js');
const { joinVoiceChannel, createAudioPlayer, createAudioResource, AudioPlayerStatus, VoiceConnectionStatus, AudioPlayer } = require('@discordjs/voice');
const ytdl = require('ytdl-core');
const { token } = require('./config.json');

const client = new Client({
    intents: [
        GatewayIntentBits.Guilds,
        GatewayIntentBits.GuildVoiceStates,
        GatewayIntentBits.GuildMessages,
        GatewayIntentBits.MessageContent,
    ],
});

client.once('ready', () => {
    console.log('Music bot is online!');
});

client.on('messageCreate', async message => {
    if (message.author.bot) return;

    const args = message.content.trim().split(' ');
    const command = args.shift().toLowerCase();

    // Join a voice channel
    if (command === '!join') {
        if (message.member.voice.channel) {
            const connection = joinVoiceChannel({
                channelId: message.member.voice.channel.id,
                guildId: message.guild.id,
                adapterCreator: message.guild.voiceAdapterCreator,
            });
            message.channel.send('Joined the voice channel!');
        } else {
            message.reply('You need to join a voice channel first!');
        }
    }

    // Play a YouTube song
    if (command === '!play') {
        if (!args.length) {
            message.reply('You need to provide a URL!');
            return;
        }

        const songUrl = args[0];
        const stream = ytdl(songUrl, { filter: 'audioonly' });
        const resource = createAudioResource(stream);

        const connection = joinVoiceChannel({
            channelId: message.member.voice.channel.id,
            guildId: message.guild.id,
            adapterCreator: message.guild.voiceAdapterCreator,
        });

        const player = createAudioPlayer();
        player.play(resource);

        connection.subscribe(player);

        message.channel.send(`Now playing: ${songUrl}`);
    }

    
