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

class GameSceen extends Phaser.Scene {
    preload() {
        // player
        this.load.spritesheet("player", "/assets/player.png", {
            frameWidth: 64,
            frameHeight: 64,
        });

        // grass tiles
        this.load.image("grass_tileset", "assets/tilesets/grass.png");
        // this.load.image("wall_tileset", "assets/tilesets/wall.png");
    }

    create() {
        // grass tiles

        const map = this.make.tilemap(TileConfig);
        // const wallTileset = map.addTilesetImage("wall_tileset");
        const grassTileset = map.addTilesetImage("grass_tileset");

        // const wallLayer = map.createBlankLayer("wall_layer", wallTileset);
        const grassLayer = map.createBlankLayer("grass_layer", grassTileset);

        grassLayer.randomize(
            0,
            0,
            mapConfig.width,
            mapConfig.height,
            [0, 50, 100]
        );

        // wallLayer.fill(1, 2, 9, mapConfig.width, 1); // Верх
        // wallLayer.fill(1, 0, mapConfig.height - 1, mapConfig.width, 1); // Низ
        // wallLayer.fill(1, 0, 1, 1, mapConfig.height - 2); // Лево
        // wallLayer.fill(1, mapConfig.width - 1, 1, 1, mapConfig.height - 2); // Право

        // wallLayer.setCollisionByProperty({ collides: true });

        this.cameras.main.setBounds(0, 0, mapConfig.width, mapConfig.height);
        this.matter.world.setBounds(0, 0, mapConfig.width, mapConfig.height);
        this.player = this.matter.add.sprite(400, 300, "player");
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
        const speed = 2;

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
