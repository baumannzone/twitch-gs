<template>
  {{ this.text }}
</template>

<script>
import { mapGetters } from "vuex";
const tmi = require("tmi.js");
import dotenv from "dotenv";

export default {
  data() {
    return {
      text: "value",
    };
  },
  created() {
    /* eslint-disable no-prototype-builtins */
    /* eslint-disable no-prototype-builtins */
    /* eslint-disable no-unused-vars */
    dotenv.config();
    console.log(this.getTwitchToken());

    const BOT_NAME = "botmannzone";
    const BAUMANNZONE = "baumannzone";
    const GUIDESMITHS = "guidesmiths";

    const client = new tmi.Client({
      options: { debug: true },
      connection: {
        reconnect: true,
      },
      identity: {
        username: BOT_NAME,
        password: process.env.CHAT_BOT_ACCESS_TOKEN || this.getTwitchToken(),
      },
      channels: [BAUMANNZONE, GUIDESMITHS],
    });

    const regexpCommand = new RegExp(/^!([a-zA-Z0-9]+)(?:\W+)?(.*)?/);

    const yesNoAnswers = [
      "Yes VoteYea",
      "MercyWing1 Of course MercyWing2",
      "Nope VoteNay",
      "I don't think so... LUL",
      "Really? MaxLOL",
      "Absolutely SeemsGood",
    ];

    const randomNumber = (min, max) =>
      Math.floor(Math.random() * (max - min)) + min;

    const love = (user) =>
      user ? `Have some love, @${user} <3` : 'Missing "@user" argument :(';

    const yesNoQuestion = (argument) =>
      argument
        ? yesNoAnswers[randomNumber(0, yesNoAnswers.length)]
        : "Where is the question? LUL";

    const commands = {
      [BAUMANNZONE]: {
        cursoplatzi: {
          response: () =>
            "Curso de Vue avanzado en platzi: https://platzi.com/cursos/avanzado-vue/ SeemsGood",
        },
        rambitonews: {
          response: () =>
            "PowerUpL Suscríbete a las Rambito News: https://www.getrevue.co/profile/baumannzone PowerUpR",
        },
        love: {
          response: (user) => love(user),
        },
        yesno: {
          response: (arg) => yesNoQuestion(arg),
        },
        comandos: {
          response: () =>
            `Comandos disponibles: ${Object.keys(commands[BAUMANNZONE])
              .map((c) => `!${c}`)
              .join(", ")}`,
        },
      },
      [GUIDESMITHS]: {
        love: {
          response: (user) => love(user),
        },
        question: {
          response: (arg) => yesNoQuestion(arg),
        },
        info: {
          response: () => "https://guidesmeetups03.eventbrite.es/",
        },
        gs: {
          response: () => "https://www.guidesmiths.com/",
        },
        dcsl: {
          response: () => "https://www.dcsl.com/",
        },
        topic: {
          response: () => `Agenda:
    💊 ".NET Pill" by Kamran Poursohrab (EN) -
    💬 "IT restart" by Sofía Sánchez (ES) -
    🙏 "God's questions" by Clara Dios (ES) -
    💬 "Digitalization in the Publishing world" by Linda de Leeuw (EN) -
    😎 "Data Science tour" - Rabadán - ES`,
        },
        commands: {
          response: () =>
            `Available commands: ${Object.keys(commands)
              .map((c) => `!${c}`)
              .join(", ")}`,
        },
      },
    };

    const simpleGreetings = [
      "vuenas",
      "vue-nas",
      "buenas",
      "hola",
      "hey",
      "hi",
      "hello",
      "alo",
      "yo",
      "y0",
    ];
    const greetings = simpleGreetings.reduce((acc, val) => {
      acc.push(
        val,
        `${val}!`,
        `${val}!!`,
        `${val}!!!`,
        "¡" + val + "!",
        "¡¡" + val + "!!"
      );
      return acc;
    }, []);

    client.connect().catch(console.error);

    client.on("connected", (addr, port) => {
      console.log(`* Connected to ${addr}:${port}`);
    });

    const checkGreetings = ({ message, channel, context }) => {
      if (greetings.includes(message.toLowerCase())) {
        client.say(channel, `${message}, @${context.username} HeyGuys`);
      }
    };

    const randomSquid = ({ channel, context }) => {
      if (randomNumber(1, 100) > 98) {
        const msg = `Hey, what's up? Squid1 Squid2 Squid3 Squid2 Squid4 @${context.username}`;
        client.say(channel, msg);
      }
    };

    client.on("message", async (channel, context, message, self) => {
      // Ignore messages from the bot
      if (self) {
        return;
      }

      // Remove first character. Channel is `#channelName`
      const cleanChannel = channel.split("#")[1];

      if (
        message.startsWith("!") &&
        message.length > 1 &&
        regexpCommand.test(message)
      ) {
        // Check commands
        const [raw, command, argument] = message.match(regexpCommand);

        if (commands[cleanChannel].hasOwnProperty(command)) {
          client.say(
            channel,
            commands[cleanChannel][command].response(argument)
          );
        }
      } else {
        checkGreetings({ message, channel, context });
        randomSquid({ channel, context });
      }
    });
  },
  methods: {
    ...mapGetters(["getTwitchToken"]),
  },
};
</script>
