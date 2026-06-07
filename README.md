# new-new-new-ultra-imporved-version-of-the-backrooms
import * as THREE from 'three';

class MonsterAI {
    constructor(player, ceilingLight, scareSound, scene) {
        // Assign necessary game objects
        this.player = player; // The player object (must have a position)
        this.ceilingLight = ceilingLight; // Light object for flickering
        this.scareSound = scareSound; // Audio object for jump scare
        this.scene = scene; // THREE.js scene for reloading if needed

        // Monster properties
        this.chaseSpeed = 3.5; // Chasing speed
        this.patrolSpeed = 2.0; // Patrol speed
        this.detectionRange = 15.0; // Light flicker range
        this.killRange = 2.0; // Jumpscare trigger range

        // State flags
        this.isJumpscaring = false;
        this.flickering = false;
        this.currentPatrolIndex = 0;

        // Patrol points (array of Vector3 positions)
        this.patrolPoints = [
            new THREE.Vector3(5, 0, 5),
            new THREE.Vector3(-5, 0, -5),
            new THREE.Vector3(-5, 0, 5),
            new THREE.Vector3(5, 0, -5),
        ];
        // Position this monster at the first patrol point
        this.position = this.patrolPoints[this.currentPatrolIndex].clone();
    }

    update(deltaTime) {
        if (this.isJumpscaring || !this.player) return;

        const distanceToPlayer = this.position.distanceTo(this.player.position);

        if (distanceToPlayer <= this.killRange) {
            this.triggerJumpscare();
        } else if (distanceToPlayer <= this.detectionRange) {
            this.chasePlayer(deltaTime);
            this.startLightFlicker();
        } else {
            this.stopLightFlicker();
            this.patrol(deltaTime);
        }
    }

    chasePlayer(deltaTime) {
        const direction = new THREE.Vector3().subVectors(this.player.position, this.position).normalize();
        this.position.add(direction.multiplyScalar(this.chaseSpeed * deltaTime));

        // Rotate to face the player
        const lookAtDirection = new THREE.Vector3(this.player.position.x, this.position.y, this.player.position.z);
        this.lookAt(lookAtDirection);
    }

    patrol(deltaTime) {
        if (this.patrolPoints.length === 0) return;

        const target = this.patrolPoints[this.currentPatrolIndex];
        const direction = new THREE.Vector3().subVectors(target, this.position).normalize();
        this.position.add(direction.multiplyScalar(this.patrolSpeed * deltaTime));

        if (this.position.distanceTo(target) < 0.5) {
            // Move to the next patrol point
            this.currentPatrolIndex = (this.currentPatrolIndex + 1) % this.patrolPoints.length;
        }
    }

    startLightFlicker() {
        if (!this.ceilingLight || this.flickering) return;

        this.flickering = true;
        this.lightFlickerInterval = setInterval(() => {
            this.ceilingLight.visible = Math.random() > 0.3; // Flicker light
        }, 200); // Flickers every 200ms
    }

    stopLightFlicker() {
        if (!this.flickering) return;

        this.flickering = false;
        clearInterval(this.lightFlickerInterval);
        if (this.ceilingLight) this.ceiling