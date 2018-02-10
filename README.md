# uglyAnts.github.io
个人简介
{
  "name": "angular-webpack",
  "version": "1.0.0",
  "description": "es6+angular+webpack",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack",
    "start": "webpack-dev-server --hot --inline --open --progress"
  },
  "keywords": [],
  "author": "shiq",
  "license": "ISC",
  "devDependencies": {
    "babel-core": "^6.26.0",
    "babel-loader": "^7.1.2",
    "babel-preset-es2015": "^6.24.1",
    "css-loader": "^0.28.8",
    "extract-text-webpack-plugin": "^3.0.2",
    "file-loader": "^1.1.6",
    "html-loader": "^0.5.4",
    "html-webpack-plugin": "^2.30.1",
    "less": "^2.7.3",
    "less-loader": "^4.0.5",
    "style-loader": "^0.19.1",
    "webpack": "^3.10.0",
    "webpack-dev-server": "^2.11.0"
  },
  "dependencies": {
    "@uirouter/angularjs": "^1.0.13",
    "angular": "^1.6.8",
    "oclazyload": "^1.1.0"
  }
}


'use strict';
// Modules
const path = require('path');
const webpack = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
// const CleanWebpackPlugin = require('clean-webpack-plugin');

module.exports = function() {
    const config = {
        entry: {
            'app':'./src/main.js',
            'vendor': ['angular', '@uirouter/angularjs']
        },
        output: {
            path: path.resolve(__dirname, 'dist'),
            publicPath: '/',
            filename: '[name].bundle.js',
            chunkFilename: '[name].bundle.js'
        },

        module: {
            loaders: [
                {
                    test: /\.js$/,
                    loader: 'babel-loader',
                    include: path.resolve(__dirname, "src")
                },
                // {
                //     test: /\.less$/,
                //     include: path.resolve(__dirname, "src"),
                //     loader: 'style-loader!css-loader!less-loader'
                // },
                {
                    test: /\.less$/,
                    use: ExtractTextPlugin.extract({
                        fallback: 'style-loader',
                        use: [{
                            loader: 'css-loader',
                            options: {
                                minimize: true // css压缩
                            }
                        }, 'less-loader']
                    })
                },
                { 
                    test: /\.html$/,
                    include: path.resolve(__dirname, "src"),
                    loader: 'html-loader' 
                },
                {
                    test: /\.(png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$/i,
                    include: path.resolve(__dirname, "src"),
                    loader: 'file-loader?name=images/[name].[ext]?[hash]'
                }
            ]
        },

        plugins: [
            new webpack.optimize.CommonsChunkPlugin({
                name: 'vendor'
            }),

            new webpack.HotModuleReplacementPlugin(),

            new HtmlWebpackPlugin({
                // chunksSortMode
                template: path.resolve(__dirname, 'src/index.html')
            }),

            // new webpack.optimize.CommonsChunkPlugin('vendor','vendor.bundle.js'),

            new ExtractTextPlugin('base.css')
        ],
        // postcss: [
        //     autoprefixer({
        //         browsers: ['last 2 version']
        //     })
        // ],
        devServer: {
            contentBase: path.join(__dirname, "src"),
            compress: true,
            hot: true,
            port: 9000
        }
    };

    if (1) {
        config.plugins.push(
            new webpack.optimize.UglifyJsPlugin({
                mangle: false
            }),
        );
    }

    return config;
}();
module.exports = function(grunt) {
    grunt.initConfig({
        pkg:grunt.file.readJSON('package.json'),

        concat: {
            options: {
                separator: ';',
                stripBanners: true,
                banner: '/*********\n项目名称：<%= pkg.name %>\n版本号：<%= pkg.version %>\n*********/\n'
            },
            dist: {
                files: {
                    'dist/common.js': ['src/vendor.bundle.js','src/app.bundle.js']
                }
            }
        },

        connect: {
            options: {
                port: 9000,
                hostname: 'localhost',
                livereload: 35729
            },
            server: {
                options: {
                    base: 'dist',
                    open:true
                }
            }
        },

        less: {
            dist: {
                options: {
                    compress: true
                },
                files: [{
                    expand: true,
                    cwd: 'src',
                    src: ['**/*.less'],
                    dest: 'dist',
                    ext: '.css'
                }]
            }
        },

        jshint: {
            files: ['src/**/*.js'],
            options: {
                globals: {
                    jQuery: true,
                    console: true,
                    module: true,
                    document: true
                }
            }
        },

        copy: {
            target: {
                files: [{
                    expand: true,
                    cwd: 'src',
                    src: [
                    '*.jpg', '**/*.jpg',
                    '*.gif', '**/*.gif',
                    '*.png', '**/*.png'],
                    dest: 'dist'
                }]
            }
        },

        htmlmin: {
            all: {
                options: {
                    removeComments: true,
                    collapseWhitespace: true
                },
                files: [{
                    expand: true,
                    cwd: 'src',
                    src: ['**/*.html','**/*.htm','**/*.ftl'],
                    dest: 'dist'
                }]
            }
        },

        uglify: {
            options: {
                // banner: '/* <%= pkg.name %> <%= grunt.template.today("dd-mm-yyyy") %> */'
                ie8:true
            },
            dist: {
                files: [{
                    expand: true,
                    cwd: 'src',
                    src: ['**/*.js','!*.min.js'],
                    dest: 'dist',
                    ext: '.js'
                }]
            }
        },

        coffee: {
            files: {
                expand: true,
                flatten: true,
                cwd: 'src',
                src: ['**/*.coffee'],
                dest: 'dist',
                ext: '.js'
            }
        },

        watch: {
            livereload: {
                options: {
                    livereload: '<%= connect.options.livereload %>'
                },
                files: ['src/**/*'],
            },
            scripts: {
                files: ['src/**/*.coffee'],
                tasks: ['coffee']
            },
            less: {
                files: ['src/**/*.less'],
                tasks: ['less']
            },
            html: {
                files: ['src/**/*.html'],
                tasks: ['htmlmin']
            }
        }
    });

    grunt.loadNpmTasks('grunt-contrib-concat');
    grunt.loadNpmTasks('grunt-contrib-jshint');
    grunt.loadNpmTasks('grunt-contrib-watch');
    grunt.loadNpmTasks('grunt-contrib-connect');
    grunt.loadNpmTasks('grunt-contrib-htmlmin');
    grunt.loadNpmTasks('grunt-contrib-copy');
    grunt.loadNpmTasks('grunt-contrib-less');
    grunt.loadNpmTasks('grunt-contrib-uglify');
    grunt.loadNpmTasks('grunt-contrib-coffee');

    grunt.registerTask('dev', ['copy','concat','less','htmlmin','connect','watch']);

    // grunt.registerTask('dev', ['jshint','copy','concat','uglify','less','htmlmin','connect','watch']);

    grunt.registerTask('build', ['jshint','copy','concat','uglify','htmlmin']);
};
