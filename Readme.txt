<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />
    <meta name="description" content="Detalles del producto" />
    <meta name="author" content="Tu Nombre" />
    <title>Detalle del Producto - Mi Tienda</title>
    <link rel="icon" type="image/x-icon" href="assets/favicon.ico" />
    <link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.5.0/font/bootstrap-icons.css" rel="stylesheet" />
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.3/dist/css/bootstrap.min.css" rel="stylesheet" />
    <link href="assets/css/styles.css" rel="stylesheet" />
    <link href="assets/css/main.css" rel="stylesheet">
    <!-- Favicons -->
    <link href="assets/img/tlalpan.jpeg" rel="icon">
    <link href="assets/img/apple-touch-icon.png" rel="apple-touch-icon">
    <style>
        .thumbnail {
            cursor: pointer;
        }
        .thumbnail img {
            max-width: 100px;
            max-height: 100px;
        }
        .zoom-icon {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 2rem;
            color: red; /* Cambia el color aquí */
            display: none;
        }
        .image-container {
            position: relative;
            display: inline-block;
        }

        
        .image-container:hover .zoom-icon {
            display: block;
        }
        .image-container:hover img {
            transform: scale(1.05); /* Efecto mini zoom */
        }
        .fullscreen-image {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.9);
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }
        .fullscreen-image img {
            max-width: 100%;
            max-height: 100%;
        }
        .fullscreen-image .close-icon {
            position: absolute;
            top: 20px;
            right: 20px;
            font-size: 2rem;
            color: white;
            cursor: pointer;
        }
    </style>
</head>

<body>
    <header id="header" class="header d-flex align-items-center fixed-top">
        <div class="container-fluid container-xl position-relative d-flex align-items-center">
    
          <a href="index.html" class="logo d-flex align-items-center me-auto">
             <img src="assets/img/tlalp.png" alt="">
            <h1 class="sitename">JLG Joyeria</h1>
          </a>
    
          <nav id="navmenu" class="navmenu">
            <ul>
              <li><a href="index.html">Regresar<br></a></li>
            </ul>
            <i class="mobile-nav-toggle d-xl-none bi bi-list"></i>
          </nav>
    
          <a class="btn-getstarted flex-md-shrink-0" href="formulario.html">Unete a nosotros</a>
    
        </div>
      </header>

      <main class="main">
        <section class="about section hero section">
            <div class="py-5 container px-4 px-lg-5 my-5">
                <div class="row gx-4 gx-lg-5 align-items-center">
                    <div class="col-md-6">
                        <div class="image-container">
                            <img id="mainImage" class="img-fluid mb-5 mb-md-0" alt="Imagen del producto" />
                            <i class="bi bi-zoom-in zoom-icon"></i>
                        </div>
                        <div class="fullscreen-image" id="fullscreenImage">
                            <img src="" alt="Imagen en pantalla completa" />
                        </div>
                    </div>
                    <div class="col-md-6">
                        <div class="small mb-1" id="productSKU"></div>
                        <h1 class="display-5 fw-bolder" id="productName"></h1>
                        <div class="fs-5 mb-5">
                            <span class="text-decoration-line-through" id="oldPrice"></span>
                            <span id="price"></span>
                        </div>
                        <p class="lead" id="productDescription"></p>
                        <div class="d-flex">
                            <input class="form-control text-center me-3" id="inputQuantity" type="num" value="1" style="max-width: 3rem" />
                            <button class="btn btn-outline-dark flex-shrink-0" type="button">
                                <i class="bi-cart-fill me-1"></i>
                                Agregar al carrito
                            </button>
                        </div>
                    </div>
                </div>
                <div class="row mt-4">
                    <div class="col-md-6">
                        <div id="thumbnailContainer" class="d-flex flex-wrap">
                            <!-- Las miniaturas se agregarán aquí -->
                        </div>
                    </div>
                </div>
            </div>
        </section>
        <section class="py-5 bg-light">
            <div class="container px-4 px-lg-5 mt-5">
                <h2 class="fw-bolder mb-4">Productos Relacionados</h2>
                <div class="row gx-4 gx-lg-5 row-cols-2 row-cols-md-3 row-cols-xl-4 justify-content-center" id="relatedProducts">
                    <!-- Productos relacionados se agregarán aquí -->
                </div>
            </div>
        </section>
        <footer class="py-5 bg-dark">
            <div class="container"><p class="m-0 text-center text-white">Derechos reservados &copy; Mi Tienda 2024</p></div>
        </footer>
        <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.2.3/dist/js/bootstrap.bundle.min.js"></script>
        <script>
            document.addEventListener('DOMContentLoaded', function () {
                const urlParams = new URLSearchParams(window.location.search);
                const productId = urlParams.get('id');
                
                fetch('products.json')
                    .then(response => response.json())
                    .then(products => {
                        const product = products.find(p => p.id == productId);
                        if (product) {
                            // Imagen principal
                            const mainImage = document.getElementById('mainImage');
                            mainImage.src = product.images[0];

                            // Miniaturas
                            const thumbnailContainer = document.getElementById('thumbnailContainer');
                            thumbnailContainer.innerHTML = product.images.map(imgSrc => `
                                <div class="thumbnail me-2 mb-2">
                                    <img src="${imgSrc}" alt="Imagen del producto" />
                                </div>
                            `).join('');

                            // Cambiar imagen principal al hacer clic en una miniatura
                            thumbnailContainer.addEventListener('click', function (event) {
                                if (event.target.tagName === 'IMG') {
                                    mainImage.src = event.target.src;
                                }
                            });

                            // Mostrar imagen en pantalla completa al hacer clic en la imagen principal
                            const fullscreenImage = document.getElementById('fullscreenImage');
                            const fullscreenImageImg = fullscreenImage.querySelector('img');
                            mainImage.addEventListener('click', function () {
                                fullscreenImageImg.src = mainImage.src;
                                fullscreenImage.style.display = 'flex';
                            });

                            // Cerrar imagen en pantalla completa al hacer clic en el icono de cerrar
                            const closeIcon = fullscreenImage.querySelector('.close-icon');
                            closeIcon.addEventListener('click', function () {
                                fullscreenImage.style.display = 'none';
                            });

                            // Otros detalles del producto
                            document.getElementById('productSKU').textContent = 'SKU: ' + product.sku;
                            document.getElementById('productName').textContent = product.name;
                            document.getElementById('oldPrice').textContent = '$' + product.oldPrice.toFixed(2);
                            document.getElementById('price').textContent = '$' + product.price.toFixed(2);
                            document.getElementById('productDescription').textContent = product.description;

                            const relatedProductsContainer = document.getElementById('relatedProducts');
                            product.related.forEach(relatedId => {
                                const relatedProduct = products.find(p => p.id == relatedId);
                                if (relatedProduct) {
                                    const relatedProductHTML = `
                                        <div class="col mb-5">
                                            <div class="card h-100">
                                                <img class="card-img-top" src="${relatedProduct.image}" alt="..." />
                                                <div class="card-body p-4">
                                                    <div class="text-center">
                                                        <h5 class="fw-bolder">${relatedProduct.name}</h5>
                                                        $${relatedProduct.price.toFixed(2)}
                                                    </div>
                                                </div>
                                                <div class="card-footer p-4 pt-0 border-top-0 bg-transparent">
                                                    <div class="text-center"><a class="btn btn-outline-dark mt-auto" href="product-details.html?id=${relatedProduct.id}">Ver producto</a></div>
                                                </div>
                                            </div>
                                        </div>
                                    `;
                                    relatedProductsContainer.insertAdjacentHTML('beforeend', relatedProductHTML);
                                }
                            });
                        }
                    })
                    .catch(error => console.error('Error cargando el archivo JSON:', error));
            });
        </script>
    </main>
</body>
</html>
