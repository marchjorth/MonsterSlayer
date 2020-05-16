<template>
    <div id="app">
        <div class="container">
            <h1>{{ title }}</h1>
            <hr />

            <div class="players">
                <div class="you">
                    <h3>You</h3>
                    <div class="healthbar">
                        <span>{{user.health}}</span>
                        <div class="health" :style="{ width: user.health + '%'}"></div>
                    </div>
                </div>

                <div class="monster">
                    <h3>Monster</h3>
                    <div class="healthbar">
                        <span>{{monster.health}}</span>
                        <div class="health" :style="{ width: monster.health + '%'}"></div>
                    </div>
                </div>
            </div>

            <div class="controls">
                <button class="btn btn__attack" @click="attack">Attack</button>
                <button
                    class="btn btn__attack--special"
                    :disabled="extras.specialCooldown == true"
                    @click="specialAttack"
                >Special attack</button>
                <button class="btn btn__heal" @click="heal">Heal</button>
                <button
                    class="btn btn__give-up"
                    @click="surrenderConfirmation"
                >{{ extras.surrenderText }}</button>
                <button
                    v-if="extras.surrender == true"
                    class="btn btn__give-up btn__give-up--surrender"
                    @click="surrender"
                >Yes</button>
            </div>

            <div class="status">
                <ul class="status__messages">
                    <template v-for="message in messages">
                        <li v-for="m in message" :key="m.id">{{ m }}</li>
                    </template>
                </ul>
            </div>
        </div>

        <section class="modal" id="modal">
            <div class="modal__dialog">
                <h2>{{ extras.result }}</h2>
                <button @click="restart()" class="btn btn__restart">{{ extras.endButtonText }}</button>
            </div>
        </section>
    </div>
</template>

<script>
export default {
    name: "app",
    data() {
        return {
            title: "Monster Slayer",
            user: {
                health: 100
            },
            monster: {
                health: 100
            },
            extras: {
                specialCooldown: false,
                moves: 0,
                surrender: false,
                surrenderText: "Surrender",
                result: "",
                endButtonText: ""
            },
            messages: []
        };
    },
    methods: {
        attack: function() {
            var userDamage = this.roll(10, 1);
            this.monster.health -= userDamage;

            var monsterDamage = this.roll(10, 1);
            this.user.health -= monsterDamage;

            this.roundStatus("dmg", userDamage, monsterDamage);

            this.extras.moves++;
            this.coolDown();
        },
        win: function() {
            var modal = document.getElementById("modal");
            if (this.user.health <= 0) {
                this.extras.result = "You lost..";
                this.extras.endButtonText = "Try again";
                modal.classList.add("active");
            } else if (this.monster.health <= 0) {
                this.extras.result = "You slayed the monster!";
                this.extras.endButtonText = "Play again";
                modal.classList.add("active");
            }
        },
        roll: function(amplifier, minAmount) {
            var randomNumber = Math.max(
                Math.floor(Math.random() * amplifier) + 1,
                minAmount
            );
            return randomNumber;
        },
        heal: function() {
            if (this.user.health != 100) {
                var healAmount = this.roll(10, 3);
                this.user.health += healAmount;

                var monsterDamage = this.roll(10, 1);
                this.user.health -= monsterDamage;

                this.extras.moves++;
                this.extras.heal = true;
                this.coolDown();

                this.roundStatus("heal", healAmount, monsterDamage);
            }
        },
        surrenderConfirmation: function() {
            this.extras.surrender = true;
            this.extras.surrenderText = "Are you sure?";
        },
        surrender: function() {
            this.monster.health = 100;
            this.user.health = 100;

            this.extras.specialCooldown = false;
            this.extras.moves = 0;
            this.messages = [];
            this.extras.surrender = false;
            this.extras.surrenderText = "Surrender";
        },
        specialAttack: function() {
            var userDamage = this.roll(20, 4);
            this.monster.health -= userDamage;

            var monsterDamage = this.roll(14, 5);
            this.user.health -= monsterDamage;

            this.extras.specialCooldown = true;
            this.extras.moves = 0;

            this.roundStatus("dmg", userDamage, monsterDamage);
        },
        coolDown: function() {
            if (this.extras.moves % 3 === 0) {
                this.extras.specialCooldown = false;
            }
        },
        roundStatus: function(type, userMessage, monsterMessage) {
            if (String(type).toLowerCase() == "heal") {
                this.messages.push(
                    {
                        message:
                            "You healed for " +
                            userMessage +
                            " hitpoints this round"
                    },
                    {
                        message:
                            "Monster did " +
                            monsterMessage +
                            " damage this round"
                    }
                );
            } else {
                this.messages.push(
                    {
                        message: "You did " + userMessage + " damage this round"
                    },
                    {
                        message:
                            "Monster did " +
                            monsterMessage +
                            " damage this round"
                    }
                );
            }

            this.win();
        },
        restart: function() {
            var modal = document.querySelector("#modal");
            modal.classList.remove("active");
            this.surrender();
        }
    }
};
</script>

<style lang="scss">
* {
    box-sizing: border-box;
    font-family: "Open Sans", sans-serif;
}

html,
body {
    padding: 0;
    margin: 0;
}

body {
    background-color: #eeeeee;
}

.container {
    max-width: 800px;
    margin: 0 auto;
    background-color: white;
    min-height: 100vh;
    padding: 0 75px;
}

h1 {
    margin: 0;
    padding: 30px 0 0;
    text-align: center;
}

hr {
    max-width: 10%;
    margin: 15px auto 15px;
}
.players {
    margin-top: 60px;
    display: flex;
    flex-direction: row;
    justify-content: space-around;
}

.players h3 {
    text-align: center;
}

.controls {
    display: flex;
    flex-direction: row;
    justify-content: center;
    margin: 60px 0;
}

.btn {
    padding: 8px 20px;
    margin: 0 10px;
    background-color: white;
    outline: 0;
    border: 2px solid black;
    font-weight: bold;
    font-size: 14px;
    transition: background-color 0.3s ease;

    &:hover {
        cursor: pointer;
        background-color: #000;
        color: #fff;
    }

    &__attack {
        border-color: #466db3;

        &:hover {
            background-color: #466db3;
        }

        &--special {
            border-color: red;
            position: relative;

            &:hover {
                background-color: red;
            }

            &[disabled] {
                border-color: grey;
                opacity: 0.5;
                pointer-events: none;

                &:before {
                    content: "cooldown..";
                    display: block;
                    position: absolute;
                    top: 40px;
                    font-size: 11px;
                    left: 50%;
                    transform: translateX(-50%);
                }
            }
        }
    }

    &__heal {
        border-color: green;

        &:hover {
            background-color: green;
        }
    }

    &__restart {
        border-color: #fff;
        color: #fff;
        background-color: transparent;

        &:hover {
            background-color: #fff;
            color: #000;
        }
    }
}

li {
    list-style-type: none;
}

.healthbar {
    border: 2px solid black;
    width: 230px;
    text-align: center;
    position: relative;
    height: 30px;

    span {
        position: absolute;
        left: 50%;
        transform: translateX(-50%);
    }
}

.health {
    width: 100%;
    height: 100%;
    background-color: green;
    transition: all 0.3s ease;
}

.status {
    &__messages {
        display: flex;
        flex-direction: column-reverse;
        padding-left: 0;

        li {
            padding: 4px 10px;
            margin: 5px 0;
            font-size: 14px;

            &:nth-child(odd) {
                background-color: #466db3;
                color: white;
                border: 1px solid #1c3867;
            }

            &:nth-child(even) {
                background-color: #ff3a3a;
                color: white;
                border: 1px solid #7b0000;
            }

            &:nth-child(2n):not(:last-of-type) {
                margin-top: 30px;
            }
        }
    }
}

.modal {
    position: fixed;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
    z-index: 101;
    background-color: rgba(0, 0, 0, 0.9);
    visibility: hidden;
    opacity: 0;
    transition: opacity 0.5s, visibility 0s 0.5s;

    &.active {
        opacity: 1;
        visibility: visible;
        transition: opacity 0.5s;

        h2 {
            color: white;
            font-size: 36px;
        }
    }

    &__dialog {
        height: 100%;
        flex-direction: column;
        display: flex;
        align-items: center;
        justify-content: center;

        .btn {
            z-index: 105;
        }
    }
}
</style>