<template>
    <nav class="navbar">
      <div class="nav-container">
        <div class="logo">Šu Šop </div>
        <ul class="nav-links">
          <li><a href="#">Home</a></li>
          <li><a href="#">Products</a></li>
          <li><a href="#">About</a></li>
          <li><a href="#">Cart</a></li>
        </ul>
      </div>
    </nav>
  </template>
  
  <style scoped>
  .navbar {
    background-color: #222;
    padding: 1rem 2rem;
    color: white;
  }
  .nav-container {
    max-width: 1200px;
    margin: auto;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .logo {
    font-size: 1.8rem;
    font-weight: bold;
  }
  .nav-links {
    list-style: none;
    display: flex;
    gap: 2rem;
    margin: 0;
  }
  .nav-links li a {
    color: white;
    text-decoration: none;
    font-size: 1.1rem;
  }
  .nav-links li a:hover {
    color: #f0c040;
  }
  </style>
  