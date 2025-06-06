<script setup>
import Phaser from "phaser";
import { onMounted } from "vue";

const ScreenConfig = {
    width: 800,
    height: 600,
};

const mapConfig = {
    width: ScreenConfig.width * 1.5,
    height: ScreenConfig.height * 1.5,
};

const TileConfig = {
    tileWidth: 16,
    tileHeight: 16,
    width: ScreenConfig.width,
    height: ScreenConfig.height,
};

let isChestOpened = false;
let isWinText = false;

class GameSceen extends Phaser.Scene {
    preload() {
        // player
        this.load.spritesheet("player", "/assets/player.png", {
            frameWidth: 64,
            frameHeight: 64,
        });
        this.load.spritesheet("chest", "/assets/chest.png", {
            frameWidth: 32,
            frameHeight: 46,
        });

        this.load.image("grass_tileset", "assets/tilesets/grass.png");
    }

    create() {
        const map = this.make.tilemap(TileConfig);
        const grassTileset = map.addTilesetImage("grass_tileset");
        const grassLayer = map.createBlankLayer("grass_layer", grassTileset);

        grassLayer.randomize(
            0,
            0,
            mapConfig.width,
            mapConfig.height,
            [0, 50, 100]
        );

        this.cameras.main.setBounds(0, 0, mapConfig.width, mapConfig.height);
        this.matter.world.setBounds(0, 0, mapConfig.width, mapConfig.height);

        this.player = this.matter.add.sprite(400, 300, "player");
        this.chest = this.matter.add.sprite(400, 200, "chest", null, {
            isStatic: true,
            isSensor: true,
        });

        this.player.setScale(1.5);
        this.chest.setScale(1.5);

        this.openChestText = this.add
            .text(400, 150, "Press [X]", {
                font: "20px Arial",
                fill: "#ffffff",
                backgroundColor: "#000000",
                padding: { x: 10, y: 5 },
            })
            .setOrigin(0.5)
            .setVisible(false);

        // this.tweens.add({
        //     targets: this.openChestText,
        //     alpha: 1,
        //     duration: 500,
        // });

        this.showText = this.add
            .text(
                this.cameras.main.centerX,
                this.cameras.main.centerY,
                "Win!\npress [X] to back",
                {
                    font: "80px Arial",
                    fill: "#ffffff",
                    backgroundColor: "#000000",
                    align: "center",
                    wordWrap: {
                        width: this.cameras.main.width * 0.8,
                        useAdvancedWrap: true,
                    },
                    padding: { x: 0, y: 200 },
                    fixedWidth: this.cameras.main.width,
                    fixedHeight: this.cameras.main.height,
                }
            )
            .setOrigin(0.5)
            .setVisible(false)
            .setScrollFactor(0);

        this.chest.setDepth(1);
        this.player.setDepth(2);
        this.openChestText.setDepth(3);
        this.showText.setDepth(4);

        this.interactionZone = this.chest.body;

        this.player.setFixedRotation();

        this.cursors = this.input.keyboard.createCursorKeys();

        this.cameras.main.startFollow(this.player);

        // player anims
        this.anims.create({
            key: "walk_up",
            frames: this.anims.generateFrameNumbers("player", {
                start: 0,
                end: 8,
            }),
            frameRate: 10,
            repeat: -1,
        });

        this.anims.create({
            key: "walk_left",
            frames: this.anims.generateFrameNumbers("player", {
                start: 9,
                end: 17,
            }),
            frameRate: 10,
            repeat: -1,
        });

        this.anims.create({
            key: "walk_down",
            frames: this.anims.generateFrameNumbers("player", {
                start: 18,
                end: 26,
            }),
            frameRate: 10,
            repeat: -1,
        });

        this.anims.create({
            key: "walk_right",
            frames: this.anims.generateFrameNumbers("player", {
                start: 27,
                end: 35,
            }),
            frameRate: 10,
            repeat: -1,
        });

        // Анимация покоя (первый кадр "вниз")
        this.anims.create({
            key: "idle",
            frames: [{ key: "player", frame: 18 }],
            frameRate: 1,
        });

        this.player.play("idle");
    }

    update() {
        const speed = 4;

        this.player.setVelocity(0);

        if (this.cursors.left.isDown) {
            this.player.setVelocityX(-speed);
            this.player.play("walk_left", true);
        } else if (this.cursors.right.isDown) {
            this.player.setVelocityX(speed);
            this.player.play("walk_right", true);
        }

        if (this.cursors.up.isDown) {
            this.player.setVelocityY(-speed);
            this.player.play("walk_up", true);
        } else if (this.cursors.down.isDown) {
            this.player.setVelocityY(speed);
            this.player.play("walk_down", true);
        }

        if (
            this.player.body.velocity.x === 0 &&
            this.player.body.velocity.y === 0
        ) {
            this.player.play("idle", true);
        }

        const distance = Phaser.Math.Distance.Between(
            this.player.x,
            this.player.y,
            this.chest.x,
            this.chest.y
        );

        let isXPressed = Phaser.Input.Keyboard.JustDown(
            this.input.keyboard.addKey("X")
        );

        if (isXPressed) {
            isWinText = false;
            this.showText.setVisible(false);
        }

        if (distance < 100) {
            if (isChestOpened) return;

            this.openChestText.setVisible(true);
            this.openChestText.setPosition(this.chest.x, this.chest.y - 75);

            if (isXPressed) {
                console.log("Клавиша X нажата!");
                isChestOpened = true;
                isWinText = true;

                this.openChestText.setVisible(false);
                this.showText.setVisible(true);
                this.chest.setFrame(1);
            }
        } else {
            isChestOpened = false;
            this.openChestText.setVisible(false);
            this.chest.setFrame(0);
        }
    }
}

onMounted(() => {
    const config = {
        type: Phaser.AUTO,
        width: ScreenConfig.width,
        height: ScreenConfig.height,
        physics: {
            default: "matter",
            matter: {
                gravity: { y: 0 },
                // debug: true,
            },
        },
        parent: "game-container",
        backgroundColor: "#000000",
        scene: [GameSceen],
    };
    const game = new Phaser.Game(config);
});
</script>

<template>
    <div id="game-container"></div>
</template>

<style>
#game-container {
    width: 800px;
    height: 600px;
}
</style>
