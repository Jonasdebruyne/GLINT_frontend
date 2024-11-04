<template>
  <div class="container">
    <header>
      <h1>GLINT</h1>
      <nav class="menu">
        <span class="menu-item active">1. SHAPE</span>
        <span class="menu-item">2. MATERIAL</span>
        <span class="menu-item">3. COLOUR</span>
        <span class="menu-item">4. OVERVIEW</span>
      </nav>
      <h1 style="visibility: hidden">GLINT</h1>
    </header>

    <div class="content">
      <div class="btns">
        <div>
          <p>Undo</p>
          <img src="../assets/icons/undo.svg" alt="Undo" />
        </div>
        <div>
          <p>Redo</p>
          <img src="../assets/icons/redo.svg" alt="Undo" />
        </div>
      </div>
      <div class="image-container"></div>
      <div class="shape-options">
        <div class="shape-option selected">
          <p class="square"></p>
          <p>Sqaure</p>
        </div>
        <div class="shape-option">
          <p class="rectangle"></p>
          <p>Rectangle</p>
        </div>
        <div class="shape-option">
          <p class="oval"></p>
          <p>Oval</p>
        </div>
        <div class="shape-option">
          <p class="circle"></p>
          <p>Circle</p>
        </div>
      </div>
    </div>

    <footer class="product-info">
      <div class="product-details">
        <div>
          <p>Product name</p>
          <h2 style="color: #aa91de" class="product-name">Bertha B602</h2>
        </div>
        <div>
          <p>Total</p>
          <h3 style="color: #aa91de" class="product-price">&euro; 209,99</h3>
        </div>
      </div>
    </footer>
  </div>
</template>

<style scoped>
.container {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  color: #fff;
  background-color: #000;
  padding: 64px;
  box-sizing: border-box;
  font-family: "Arial, sans-serif";
}

header {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  padding: 0 40px;
  margin-bottom: 40px;
}

header h1 {
  font-size: 1.5rem;
  color: #ffffff;
  letter-spacing: 1px;
}

.menu {
  display: flex;
  gap: 30px;
  margin: 0 auto;
  text-align: center;
}

.menu-item {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.4);
  cursor: pointer;
  transition: color 0.3s;
}

.menu-item:hover,
.menu-item.active {
  color: #aa91de;
}

.content {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  flex-grow: 1;
  padding: 0 2rem;
  width: 100%;
}

.btns {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.btns div {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 0.5rem;
}

.btns div img {
  width: 32px;
  height: 32px;
}

.image-container {
  background-image: url("../assets/images/testGlasses.png");
  background-position: center;
  background-repeat: no-repeat;
  background-size: contain;
  width: 100%;
  height: 300px;
}

.shape-options {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.shape-option {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}

.shape-option .square {
  width: 64px;
  height: 64px;
  border: 4px solid #a2a2a2;
  background-color: transparent;
}

.shape-option .oval {
  width: 82px;
  height: 64px;
  background-color: transparent;
  border: 4px solid #a2a2a2;
  border-radius: 50%;
}

.shape-option .circle {
  width: 64px;
  height: 64px;
  background-color: transparent;
  border: 4px solid #a2a2a2;
  border-radius: 50%;
}

.shape-option .rectangle {
  width: 91px;
  height: 64px;
  border: 4px solid #a2a2a2;
  background-color: transparent;
}

.shape-label {
  color: #888;
  font-size: 1.1rem;
}

.shape-option.selected p {
  border-color: #aa91de;
  color: #aa91de;
}

.product-info {
  width: 100%;
  padding: 20px 40px;
  box-sizing: border-box;
}

.product-details {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.product-details div {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  font-size: 1rem;
}

.product-name {
  color: #fff;
}

.product-price {
  color: #aa91de;
  font-size: 2.5rem;
}
</style>
