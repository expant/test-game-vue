<script setup>
import Phaser from "phaser";
import { onMounted } from "vue";

const ScreenConfig = {
    width: 800,
    height: 600,
};

const TileConfig = {
    tileWidth: 16,
    tileHeight: 16,
    width: ScreenConfig.width,
    height: ScreenConfig.height,
};

class GameSceen extends Phaser.Scene {
    preload() {
        // player
        this.load.spritesheet("player", "/assets/player.png", {
            frameWidth: 64,
            frameHeight: 64,
        });

        // grass tiles
        this.load.image("grass_tileset", "assets/grass_tileset.png");
    }

    create() {
        // grass tiles
        const map = this.make.tilemap(TileConfig);
        const tileset = map.addTilesetImage("grass_tileset");
        const layer = map.createBlankLayer("grass_layer", tileset);

        layer.randomize(
            0,
            0,
            ScreenConfig.width / 16,
            ScreenConfig.height / 16,
            [0, 50, 100]
        );

        // player anims
        this.player = this.matter.add.sprite(400, 300, "player");
        this.cursors = this.input.keyboard.createCursorKeys();

        this.cameras.main.startFollow(this.player);

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
            frames: [{ key: "player", frame: 0 }],
            frameRate: 1,
        });

        this.player.play("idle");
    }

    update() {
        const speed = 1;

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
                debug: true,
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
