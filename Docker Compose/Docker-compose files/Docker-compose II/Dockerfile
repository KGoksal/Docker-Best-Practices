# Create your custom Drupal image here, based on the official Drupal image
FROM drupal:9

# Update package lists, install Git, and clean up to reduce image size
RUN apt-get update && apt-get install -y git \
    && rm -rf /var/lib/apt/lists/*

# Set the working directory to the Drupal themes directory
WORKDIR /var/www/html/themes

# Clone the Bootstrap theme from the specified branch, and set proper ownership
RUN git clone --branch 8.x-4.x --single-branch --depth 1 https://git.drupalcode.org/project/bootstrap.git \
    && chown -R www-data:www-data bootstrap

# Change the working directory back to the Drupal root directory
WORKDIR /var/www/html
