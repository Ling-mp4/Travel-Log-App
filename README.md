package com.example.myapplication

import android.net.Uri
import android.os.Bundle
import android.util.Log
import androidx.activity.ComponentActivity
import androidx.activity.compose.rememberLauncherForActivityResult
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.foundation.Image
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.layout.width
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.verticalScroll
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.automirrored.filled.ArrowBack
import androidx.compose.material.icons.automirrored.filled.List
import androidx.compose.material.icons.filled.AccountCircle
import androidx.compose.material.icons.filled.Add
import androidx.compose.material.icons.filled.Delete
import androidx.compose.material.icons.filled.Favorite
import androidx.compose.material.icons.filled.FavoriteBorder
import androidx.compose.material.icons.filled.Home
import androidx.compose.material3.Button
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.NavigationBar
import androidx.compose.material3.NavigationBarItem
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Slider
import androidx.compose.material3.Text
import androidx.compose.material3.TopAppBar
import androidx.compose.runtime.Composable
import androidx.compose.runtime.DisposableEffect
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.SideEffect
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableIntStateOf
import androidx.compose.runtime.mutableStateListOf
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.saveable.rememberSaveable
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.vector.ImageVector
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewmodel.compose.viewModel
import androidx.navigation.NavController
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import coil.compose.rememberAsyncImagePainter

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            App()
        }
    }
}

@Composable
fun App() {
    val navController = rememberNavController()
    val itemViewModel: ItemViewModel = viewModel()

    val navItemList = listOf(
        NavItem("Home", icon = Icons.Default.Home, Screen.Home.route),
        NavItem("Item List", icon = Icons.AutoMirrored.Filled.List, Screen.ItemList.route),
        NavItem("Notes", icon = Icons.Default.AccountCircle, Screen.Personal.route)
    )

    var selectedIndex by rememberSaveable { mutableIntStateOf(0) }

    Scaffold (
        modifier = Modifier
            .fillMaxSize(),
        bottomBar = {
            NavigationBar {
                navItemList.forEachIndexed{index, item ->
                    NavigationBarItem(
                        selected = selectedIndex == index,
                        onClick = {
                            selectedIndex = index
                            if (navController.currentDestination?.route != item.route) {
                                navController.navigate(item.route) {
                                    launchSingleTop = true
                                    restoreState = true
                                }
                            }
                                  },
                        icon = { Icon(imageVector = item.icon, contentDescription = item.title)},
                        label = {Text(text = item.title)}
                    )
                }
            }

        }
    ) {
        innerPadding ->
        NavHost(
            navController = navController,
            startDestination = Screen.Home.route,
            modifier = Modifier
                .fillMaxSize()
                .padding(innerPadding)
        ) {
            composable(Screen.Home.route) {HomeScreen()}
            composable(Screen.ItemList.route) {ItemListScreen(navController = navController, viewModel = itemViewModel)}
            composable(Screen.ItemDetail.route) {ItemDetailScreen(navController, viewModel = itemViewModel)}
            composable(Screen.Personal.route) {PersonalScreen(viewModel = itemViewModel)}
        }
    }

}

sealed class Screen(val route: String) {
    object Home: Screen("home")
    object ItemList: Screen("itemList")
    object ItemDetail: Screen("itemDetail")
    object Personal: Screen("personal")
}

data class NavItem(
    val title: String,
    val icon: ImageVector,
    val route: String
)


@Composable
fun HomeScreen() {

    Column(
        modifier = Modifier.fillMaxSize(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally

    ) {
        Row {
            Text(
                text = "Travel Logs!",
                fontSize = 36.sp
            )
        }
        Row (
            modifier = Modifier
                .fillMaxWidth()
                .padding(8.dp),
            horizontalArrangement = Arrangement.Center
        ) {
        Text(
            text = "This app is designed to save your travel logs alongside your favorite " +
                    "moment to help remember them!",
            fontSize = 18.sp,
            textAlign = TextAlign.Center
        )
    }
    }

}
@Composable
fun ItemListScreen(navController: NavController, viewModel: ItemViewModel) {
    val items = viewModel.items

    Column (
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        LazyColumn (
            modifier = Modifier
                .weight(1f)
        ) {
            items(items) {
                item -> Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(vertical = 8.dp)
                    .clickable {
                        viewModel.selectItem(item)
                        navController.navigate("itemDetail")
                    },
                    horizontalArrangement = Arrangement.spacedBy(16.dp)
                ) {
                Image(
                    painter = painterResource(id = item.imageRes),
                    contentDescription = item.name,
                    modifier = Modifier
                        .width(80.dp)
                        .height(80.dp),
                    contentScale = ContentScale.Crop

                )
                Text(text = item.name, fontSize = 20.sp)
                }
            }
        }
        Row(
            modifier = Modifier
                .fillMaxWidth(),
            horizontalArrangement = Arrangement.Center
        ) {
            IconButton(
                onClick = {
                    navController.navigate("personal")
                }
            ) {
                Icon(
                    imageVector = Icons.Default.Add,
                    contentDescription = "Add"
                )
                Spacer(
                    modifier = Modifier
                        .width(4.dp)
                )
                Text(
                    text = "Add New"
                )
            }
        }
    }
}

data class Item(
    val id: Int,
    val name: String,
    val imageRes: Int
)

data class ItemNote(
    val item: Item,
    val note: String,
    val imageUri: Uri? = null,
    val rating: Int = 0
)

class ItemViewModel : ViewModel() {

    var items = mutableStateListOf<Item>()

    init {
        val drawableClass = R.drawable::class.java
        val fields = drawableClass.fields

        var counter = 1
        for (field in fields) {
            try {
                val resId = field.getInt(null)
                val name = field.name
                if (!name.contains("ic_launcher")) {
                    items.add(Item(
                        counter++,
                        name.split("_")
                            .joinToString(" ") {it.replaceFirstChar { c -> c.uppercase()}},
                        resId))
                }
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }
    }

    var selectedItem = mutableStateOf<Item?>(null)
    var notes = mutableStateListOf<ItemNote>()

    fun selectItem(item: Item) {
        selectedItem.value = item
    }
    fun saveNoteForItem(noteText: String, imageUri: Uri? = null, rating: Int = 0) {
        val current = selectedItem.value ?: return
        notes.add(ItemNote(current, noteText, imageUri, rating))
    }

    fun deleteNote(note: ItemNote) {
        notes.remove(note)
    }
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ItemDetailScreen(navController: NavController, viewModel: ItemViewModel) {
    val item = viewModel.selectedItem.value
    if (item == null) {
        Box(
            modifier = Modifier
                .fillMaxSize(),
            contentAlignment = Alignment.Center
        ) {
            Text(
                text = "No item selected", fontSize = 20.sp
            )
        }
        return
    }
    var noteText by rememberSaveable { mutableStateOf("") }
    var selectedImageUri by rememberSaveable { mutableStateOf<Uri?>(null) }
    var rating by rememberSaveable {mutableStateOf(0f)}

    LaunchedEffect(item) {
        Log.d("LifeCycle", "Opened details for ${item.name}")
    }
    SideEffect {
        Log.d("Lifecycle", "Recomposed ItemDetailScreen ${item.name}")
    }

    DisposableEffect(Unit) {
        Log.d("Lifecycle", "ItemDetailScreen composed for ${item.name}")
        onDispose {
            Log.d("Lifecycle", "ItemDetailScreen disposed for ${item.name}")
        }
    }

    val launcher = rememberLauncherForActivityResult(
        contract = ActivityResultContracts.GetContent()
    ) {uri: Uri? ->
        selectedImageUri = uri
    }

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Item Details") },
                navigationIcon = {
                    IconButton(onClick = { navController.popBackStack() }) {
                        Icon(
                            imageVector = Icons.AutoMirrored.Filled.ArrowBack,
                            contentDescription = "Back"
                        )
                    }
                }
            )
        }
    ) { innerPadding ->
        Box(
            modifier = Modifier
                .fillMaxSize()
                .padding(innerPadding),
            contentAlignment = Alignment.Center
        ) {
            Column(
                horizontalAlignment = Alignment.CenterHorizontally,
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(16.dp)
                    .verticalScroll(rememberScrollState())
            ) {
                Text(
                    text = item.name,
                    fontSize = 36.sp
                )
                Spacer(
                    modifier = Modifier
                        .height(16.dp)
                )
                Image(
                    painter = painterResource(id = item.imageRes),
                    contentDescription = item.name,
                    modifier = Modifier
                        .fillMaxWidth()
                        .height(300.dp),
                    contentScale = ContentScale.Crop
                )
                Spacer(
                    modifier = Modifier
                        .height(16.dp)
                )

                OutlinedTextField(
                    value = noteText,
                    onValueChange = { noteText = it },
                    label = { Text("Enter a note") },
                    modifier = Modifier
                        .fillMaxWidth()
                )
                Spacer(
                    modifier = Modifier
                        .height(16.dp)
                )

                Text("Rating!", fontSize = 20.sp)

                Spacer(
                    modifier = Modifier
                        .height(8.dp)
                )

                Slider(
                    value = rating,
                    onValueChange = {rating = it},
                    valueRange = 0f..5f,
                    steps = 4,
                    modifier = Modifier
                        .fillMaxWidth(1f)

                )
                Row(
                    horizontalArrangement = Arrangement.spacedBy(16.dp)
                ) {
                    repeat(5) {index ->
                        val heartFilled = index < rating.toInt()
                        Icon(
                            imageVector = if (heartFilled) {
                                Icons.Default.Favorite
                            } else {
                                Icons.Default.FavoriteBorder
                            },
                            contentDescription = "Star $index",
                            modifier = Modifier
                                .size(32.dp)
                        )
                    }
                }

                Spacer(
                    modifier = Modifier
                        .height(16.dp)
                )

                selectedImageUri?.let {
                    Image(
                        painter = rememberAsyncImagePainter(it),
                        contentDescription = "Selected image",
                        modifier = Modifier
                            .fillMaxWidth()
                            .height(200.dp)
                            .padding(vertical = 8.dp),
                        contentScale = ContentScale.Crop
                    )
                }

                Button(onClick = {launcher.launch("image/*")}) {
                    Text("Add Image")
                }

                Button(
                    onClick = {
                        viewModel.saveNoteForItem(noteText, selectedImageUri, rating.toInt())
                        navController.navigate("personal")
                    }
                ) {
                    Text(
                        "Save Note"
                    )
                }

            }
        }
    }
}

@Composable
fun PersonalScreen(viewModel: ItemViewModel) {
    val notes = viewModel.notes
    var search by rememberSaveable { mutableStateOf("") }

    val filteredNotes = notes.filter {
        it.note.contains(search, ignoreCase = true) || it.item.name.contains(search, ignoreCase = true)
    }

    LaunchedEffect(search) {
        Log.d("Lifecycle", "Search changed: $search")
    }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(
            text = "Personal Notes",
            fontSize = 36.sp
        )
        Spacer(
            modifier = Modifier
            .height(16.dp)
        )

        OutlinedTextField(
            value = search,
            onValueChange = {search = it},
            label = {
                Text(text = "Search for notes")
            },
                modifier = Modifier.fillMaxWidth()

        )

        Spacer(
            modifier = Modifier
                .height(16.dp)
        )
        if (filteredNotes.isEmpty()) {
            Text("\nNo notes found.")
        } else {
            LazyColumn (
                modifier = Modifier.fillMaxWidth()
            ) {
                items(filteredNotes.size) {
                    index ->
                    val note = filteredNotes[index]
                    Row (
                        modifier = Modifier
                            .fillMaxWidth()
                            .padding(vertical = 8.dp),
                        verticalAlignment = Alignment.CenterVertically,
                        horizontalArrangement = Arrangement.SpaceBetween
                    ) {
                        if (note.imageUri != null) {
                            Image(
                                painter = rememberAsyncImagePainter(note.imageUri),
                                contentDescription = note.item.name,
                                modifier = Modifier
                                    .height(80.dp)
                                    .width(80.dp)
                                    .padding(end = 16.dp),
                                contentScale = ContentScale.Crop
                            )
                        } else {
                            Image(
                                painter = painterResource(id = note.item.imageRes),
                                contentDescription = note.item.name,
                                modifier = Modifier
                                    .height(80.dp)
                                    .width(80.dp)
                                    .padding(end = 16.dp),
                                contentScale = ContentScale.Crop
                            )
                        }
                        Column {
                            Text(
                                text = note.item.name,
                                fontSize = 20.sp
                            )
                            Text(
                                text = note.note,
                                fontSize = 16.sp
                            )
                        }
                        Row(
                            modifier = Modifier
                                .fillMaxWidth(),
                            horizontalArrangement = Arrangement.End
                        ) {
                            Text(
                                text = "${note.rating}"
                            )
                            Icon(
                                imageVector = Icons.Default.Favorite,
                                contentDescription = "Rating"
                            )
                            
                            IconButton(onClick = {
                                viewModel.deleteNote(note)
                            }) {
                                Icon(
                                    imageVector = Icons.Default.Delete,
                                    contentDescription = "Delete"
                                )
                            }
                        }
                    }
                }
            }

        }
    }
}


