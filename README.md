# suarezplayer
lecteur iptv
01. package.json
====================================================================
{
  "name": "suarezplayer",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "expo start",
    "android": "expo run:android",
    "web": "expo start --web"
  },
  "dependencies": {
    "expo": "~52.0.0",
    "react": "18.3.1",
    "react-native": "0.76.0",
    "react-native-gesture-handler": "~2.20.0",
    "react-native-safe-area-context": "4.12.0",
    "zustand": "^5.0.0",
    "@realm/react": "^0.11.0",
    "realm": "^12.12.0",
    "expo-video": "~2.0.0",
    "react-native-toast-message": "^2.2.0",
    "@react-navigation/native": "^6.1.0",
    "@react-navigation/stack": "^6.3.0",
    "@react-navigation/bottom-tabs": "^6.5.0",
    "@react-native-async-storage/async-storage": "1.23.0"
  },
  "devDependencies": {
    "@types/react": "~18.3.0",
    "typescript": "^5.6.0"
  }
}

====================================================================
02. App.tsx
====================================================================
import React from 'react';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import Toast from 'react-native-toast-message';
import { RealmProvider } from '@realm/react';
import AppNavigator from './src/navigation/AppNavigator';

export default function App() {
  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      <SafeAreaProvider>
        <RealmProvider>
          <AppNavigator />
          <Toast />
        </RealmProvider>
      </SafeAreaProvider>
    </GestureHandlerRootView>
  );
}

====================================================================
03. src/navigation/AppNavigator.tsx
====================================================================
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import BottomTabNavigator from './BottomTabNavigator';
import LoginScreen from '../screens/LoginScreen';
import VodSeriesScreen from '../screens/VodSeriesScreen';

const Stack = createStackNavigator();

export default function AppNavigator() {
  return (
    <NavigationContainer>
      <Stack.Navigator screenOptions={{ headerShown: false }}>
        <Stack.Screen name="Login" component={LoginScreen} />
        <Stack.Screen name="Main" component={BottomTabNavigator} />
        <Stack.Screen name="Series" component={VodSeriesScreen} options={{ headerShown: true, title: 'Série' }} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

====================================================================
04. src/navigation/BottomTabNavigator.tsx
====================================================================
import React from 'react';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import PlayerScreen from '../screens/PlayerScreen';
import CategoriesScreen from '../screens/CategoriesScreen';
import VodCatalogScreen from '../screens/VodCatalogScreen';
import EPGGridView from '../components/EPGGridView';
import SettingsScreen from '../screens/SettingsScreen';

const Tab = createBottomTabNavigator();

export default function BottomTabNavigator() {
  return (
    <Tab.Navigator
      screenOptions={{
        tabBarStyle: { backgroundColor: '#1a1a2e', borderTopWidth: 0 },
        tabBarActiveTintColor: '#fff',
        tabBarInactiveTintColor: '#888',
        headerStyle: { backgroundColor: '#1a1a2e' },
        headerTintColor: '#fff',
      }}
    >
      <Tab.Screen name="Accueil" component={PlayerScreen} options={{ tabBarLabel: '📺 En direct' }} />
      <Tab.Screen name="Catégories" component={CategoriesScreen} options={{ tabBarLabel: '📂 Catégories' }} />
      <Tab.Screen name="Catalogue" component={VodCatalogScreen} options={{ tabBarLabel: '🎬 VOD' }} />
      <Tab.Screen name="GuideTV" component={EPGGridView} options={{ tabBarLabel: '📅 EPG' }} />
      <Tab.Screen name="Paramètres" component={SettingsScreen} options={{ tabBarLabel: '⚙️ Réglages' }} />
    </Tab.Navigator>
  );
}

====================================================================
05. src/screens/PlayerScreen.tsx
====================================================================
import React, { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet, FlatList, TouchableOpacity } from 'react-native';
import { fetchM3U } from '../utils/m3uParser';
import VideoPlayer from '../components/player/VideoPlayer';
import SearchBar from '../components/SearchBar';
import { usePlayerStore } from '../store/playerStore';
import { useFavoritesStore } from '../store/favoritesStore';
import FavoritesScreen from './FavoritesScreen';

type TabType = 'liste' | 'favoris';

export default function PlayerScreen() {
  const [url, setUrl] = useState('');
  const [channels, setChannels] = useState<any[]>([]);
  const [selectedUrl, setSelectedUrl] = useState<string | null>(null);
  const [searchQuery, setSearchQuery] = useState('');
  const [activeTab, setActiveTab] = useState<TabType>('liste');
  const { setUrl: setPlayerUrl } = usePlayerStore();
  const { toggleFavorite, isFavorite } = useFavoritesStore();

  const loadPlaylist = async () => {
    try {
      const list = await fetchM3U(url);
      setChannels(list.map((c, i) => ({ ...c, id: `ch-${i}` })));
    } catch (err) {
      alert('Erreur de chargement — vérifiez le lien');
    }
  };

  const playChannel = (channel: any) => {
    setSelectedUrl(channel.url);
    setPlayerUrl(channel.url);
  };

  const filteredChannels = channels.filter(c =>
    c.name.toLowerCase().includes(searchQuery.toLowerCase())
  );

  return (
    <View style={styles.container}>
      <Text style={styles.title}>📺 SuarezPlayer</Text>
      
      <VideoPlayer streamUrl={selectedUrl} />
      
      <View style={styles.tabs}>
        <TouchableOpacity
          style={[styles.tab, activeTab === 'liste' && styles.tabActive]}
          onPress={() => setActiveTab('liste')}
        >
          <Text style={[styles.tabText, activeTab === 'liste' && styles.tabTextActive]}>Toutes</Text>
        </TouchableOpacity>
        <TouchableOpacity
          style={[styles.tab, activeTab === 'favoris' && styles.tabActive]}
          onPress={() => setActiveTab('favoris')}
        >
          <Text style={[styles.tabText, activeTab === 'favoris' && styles.tabTextActive]}>❤️</Text>
        </TouchableOpacity>
      </View>
      
      {activeTab === 'liste' ? (
        <>
          <TextInput
            style={styles.input}
            placeholder="Lien M3U"
            value={url}
            onChangeText={setUrl}
          />
          <Button title="Charger la playlist" onPress={loadPlaylist} />
          
          <SearchBar
            value={searchQuery}
            onChange={setSearchQuery}
            placeholder="Rechercher une chaîne..."
          />
          
          <FlatList
            data={filteredChannels}
            keyExtractor={(item) => item.id}
            style={{ flex: 1 }}
            renderItem={({ item }) => (
              <View style={styles.channelRow}>
                <TouchableOpacity
                  style={[
                    styles.channel,
                    selectedUrl === item.url && styles.channelActive
                  ]}
                  onPress={() => playChannel(item)}
                >
                  <Text style={styles.channelName}>{item.name}</Text>
                </TouchableOpacity>
                <TouchableOpacity
                  style={styles.favBtn}
                  onPress={() => toggleFavorite(item)}
                >
                  <Text style={styles.favIcon}>{isFavorite(item.id) ? '⭐' : '☆'}</Text>
                </TouchableOpacity>
              </View>
            )}
          />
        </>
      ) : (
        <FavoritesScreen onSelectChannel={playChannel} />
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, backgroundColor: '#121212' },
  title: { fontSize: 24, fontWeight: 'bold', color: '#fff', marginBottom: 12, textAlign: 'center' },
  tabs: { flexDirection: 'row', marginBottom: 12 },
  tab: { flex: 1, paddingVertical: 10, alignItems: 'center', borderBottomWidth: 2, borderBottomColor: '#333' },
  tabActive: { borderBottomColor: '#1a5fb4' },
  tabText: { color: '#888' },
  tabTextActive: { color: '#fff', fontWeight: '600' },
  input: { backgroundColor: '#fff', padding: 10, borderRadius: 6, marginBottom: 12 },
  channelRow: {
    flexDirection: 'row',
    alignItems: 'center',
    gap: 8,
    marginBottom: 6,
  },
  channel: {
    flex: 1,
    padding: 14,
    backgroundColor: '#1a1a1a',
    borderRadius: 6,
  },
  channelActive: { backgroundColor: '#1a5fb4' },
  channelName: { color: '#fff', fontSize: 15 },
  favBtn: { padding: 6 },
  favIcon: { fontSize: 20 },
});

====================================================================
06. src/screens/VodCatalogScreen.tsx
====================================================================
import React, { useEffect, useState } from 'react';
import { View, Text, FlatList, TouchableOpacity, StyleSheet, Image } from 'react-native';
import { useNavigation } from '@react-navigation/native';

interface VodItem {
  id: string;
  title: string;
  thumbnail: string;
  type: 'movie' | 'series';
  description: string;
}

export default function VodCatalogScreen() {
  const [items, setItems] = useState<VodItem[]>([]);
  const [filter, setFilter] = useState<'all' | 'movie' | 'series'>('all');
  const navigation = useNavigation();

  useEffect(() => {
    loadMockData();
  }, []);

  const loadMockData = () => {
    setItems([
      {
        id: '1',
        title: 'Film Exemple',
        thumbnail: 'https://via.placeholder.com/150x200/1a1a2e/ffffff?text=Film+1',
        type: 'movie',
        description: 'Un film disponible en streaming',
      },
      {
        id: '2',
        title: 'Série Exemple',
        thumbnail: 'https://via.placeholder.com/150x200/1a5fb4/ffffff?text=Série+1',
        type: 'series',
        description: 'Saison 1 — 10 épisodes',
      },
    ]);
  };

  const filtered = filter === 'all' ? items : items.filter(i => i.type === filter);

  return (
    <View style={styles.container}>
      <Text style={styles.title}>🎬 Catalogue VOD</Text>
      
      <View style={styles.filters}>
        {(['all', 'movie', 'series'] as const).map(f => (
          <TouchableOpacity
            key={f}
            style={[styles.btn, filter === f && styles.btnActive]}
            onPress={() => setFilter(f)}
          >
            <Text style={[styles.btnText, filter === f && styles.btnTextActive]}>
              {f === 'all' ? 'Tout' : f === 'movie' ? '🎥 Films' : '📺 Séries'}
            </Text>
          </TouchableOpacity>
        ))}
      </View>

      <FlatList
        data={filtered}
        numColumns={2}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <TouchableOpacity
            style={styles.card}
            onPress={() => {
              if (item.type === 'series') {
                navigation.navigate('Series', { seriesId: item.id } as any);
              }
            }}
          >
            <Image source={{ uri: item.thumbnail }} style={styles.thumb} />
            <Text style={styles.itemTitle} numberOfLines={2}>{item.title}</Text>
            <Text style={styles.desc} numberOfLines={1}>{item.description}</Text>
          </TouchableOpacity>
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, backgroundColor: '#121212' },
  title: { fontSize: 22, fontWeight: 'bold', color: '#fff', marginBottom: 12 },
  filters: { flexDirection: 'row', gap: 8, marginBottom: 16 },
  btn: { paddingVertical: 6, paddingHorizontal: 12, borderRadius: 20, backgroundColor: '#2a2a2a' },
  btnActive: { backgroundColor: '#1a5fb4' },
  btnText: { color: '#ccc' },
  btnTextActive: { color: '#fff', fontWeight: '600' },
  card: { flex: 1, margin: 8, backgroundColor: '#1e1e1e', borderRadius: 10, overflow: 'hidden' },
  thumb: { width: '100%', height: 150, backgroundColor: '#333' },
  itemTitle: { color: '#fff', padding: 8, fontWeight: '600' },
  desc: { color: '#888', paddingHorizontal: 8, paddingBottom: 8, fontSize: 12 },
});

====================================================================
07. src/screens/VodSeriesScreen.tsx
====================================================================
import React from 'react';
import { View, Text, FlatList, TouchableOpacity, StyleSheet } from 'react-native';

const seasons = [
  {
    id: 's1',
    name: 'Saison 1',
    episodes: [
      { id: 'e1', title: 'Épisode 1 — Le début', duration: '45 min' },
      { id: 'e2', title: 'Épisode 2 — Le voyage', duration: '42 min' },
    ],
  },
  {
    id: 's2',
    name: 'Saison 2',
    episodes: [
      { id: 'e1', title: 'Épisode 1 — Nouveaux départs', duration: '44 min' },
    ],
  },
];

export default function VodSeriesScreen() {
  const [expandedSeason, setExpandedSeason] = React.useState<string | null>('s1');

  return (
    <View style={styles.container}>
      <Text style={styles.title}>📺 Série</Text>
      
      <FlatList
        data={seasons}
        keyExtractor={(s) => s.id}
        renderItem={({ item: season }) => (
          <View style={styles.seasonBlock}>
            <TouchableOpacity
              style={styles.seasonHeader}
              onPress={() => setExpandedSeason(expandedSeason === season.id ? null : season.id)}
            >
              <Text style={styles.seasonName}>{season.name}</Text>
              <Text style={styles.toggle}>{expandedSeason === season.id ? '▼' : '▶'}</Text>
            </TouchableOpacity>
            
            {expandedSeason === season.id && (
              <FlatList
                data={season.episodes}
                keyExtractor={(ep) => ep.id}
                renderItem={({ item: ep }) => (
                  <TouchableOpacity style={styles.episode}>
                    <Text style={styles.epTitle}>{ep.title}</Text>
                    <Text style={styles.epDuration}>{ep.duration}</Text>
                  </TouchableOpacity>
                )}
              />
            )}
          </View>
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, backgroundColor: '#121212' },
  title: { fontSize: 22, fontWeight: 'bold', color: '#fff', marginBottom: 16 },
  seasonBlock: { marginBottom: 16, backgroundColor: '#1e1e1e', borderRadius: 8, overflow: 'hidden' },
  seasonHeader: { flexDirection: 'row', justifyContent: 'space-between', padding: 14 },
  seasonName: { color: '#fff', fontWeight: '600', fontSize: 16 },
  toggle: { color: '#888' },
  episode: { padding: 12, paddingLeft: 24, borderTopWidth: 1, borderTopColor: '#333' },
  epTitle: { color: '#fff' },
  epDuration: { color: '#888', fontSize: 12, marginTop: 2 },
});

====================================================================
08. src/screens/FavoritesScreen.tsx
====================================================================
import React from 'react';
import { View, Text, FlatList, TouchableOpacity, StyleSheet } from 'react-native';
import { useFavoritesStore } from '../store/favoritesStore';

interface Props {
  onSelectChannel: (channel: any) => void;
}

export default function FavoritesScreen({ onSelectChannel }: Props) {
  const { items, toggleFavorite } = useFavoritesStore();

  if (items.length === 0) {
    return (
      <View style={styles.emptyContainer}>
        <Text style={styles.emptyText}>❤️</Text>
        <Text style={styles.emptySubtitle}>Pas encore de favoris</Text>
        <Text style={styles.emptyHint}>Cliquez sur ⭐ depuis la liste pour ajouter</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>❤️ Mes Favoris</Text>
      
      <FlatList
        data={items}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <TouchableOpacity
            style={styles.item}
            onPress={() => onSelectChannel(item)}
          >
            <View style={styles.itemInfo}>
              <Text style={styles.itemName}>{item.name}</Text>
              {item.group && <Text style={styles.itemGroup}>{item.group}</Text>}
            </View>
            <TouchableOpacity
              onPress={(e) => {
                e.stopPropagation();
                toggleFavorite(item);
              }}
            >
              <Text style={styles.removeBtn}>⭐</Text>
            </TouchableOpacity>
          </TouchableOpacity>
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, backgroundColor: '#121212' },
  title: { fontSize: 22, fontWeight: 'bold', color: '#fff', marginBottom: 16 },
  emptyContainer: { flex: 1, justifyContent: 'center', alignItems: 'center', backgroundColor: '#121212' },
  emptyText: { fontSize: 48, marginBottom: 16 },
  emptySubtitle: { color: '#fff', fontSize: 18, marginBottom: 8 },
  emptyHint: { color: '#888', fontSize: 14 },
  item: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    padding: 14,
    backgroundColor: '#1e1e1e',
    borderRadius: 8,
    marginBottom: 8,
  },
  itemInfo: { flex: 1 },
  itemName: { color: '#fff', fontSize: 15 },
  itemGroup: { color: '#888', fontSize: 12, marginTop: 2 },
  removeBtn: { fontSize: 20 },
});

====================================================================
09. src/screens/CategoriesScreen.tsx
====================================================================
import React, { useState } from 'react';
import { View, Text, FlatList, TouchableOpacity, StyleSheet } from 'react-native';

interface Category {
  id: string;
  name: string;
  icon?: string;
  count?: number;
}

const defaultCategories: Category[] = [
  { id: 'all', name: 'Toutes les chaînes', icon: '📺' },
  { id: 'sports', name: 'Sports', icon: '⚽' },
  { id: 'news', name: 'Actualités', icon: '📰' },
  { id: 'movies', name: 'Films', icon: '🎥' },
  { id: 'entertainment', name: 'Divertissement', icon: '🎭' },
  { id: 'kids', name: 'Jeunesse', icon: '👶' },
  { id: 'documentary', name: 'Documentaires', icon: '🌍' },
  { id: 'music', name: 'Musique', icon: '🎵' },
];

interface Props {
  onSelectCategory: (categoryId: string) => void;
  customCategories?: Category[];
}

export default function CategoriesScreen({ onSelectCategory, customCategories }: Props) {
  const categories = customCategories || defaultCategories;
  const [selectedId, setSelectedId] = useState<string | null>(null);

  const handleSelect = (id: string) => {
    setSelectedId(id);
    onSelectCategory(id);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>📂 Catégories</Text>
      
      <FlatList
        data={categories}
        numColumns={2}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <TouchableOpacity
            style={[styles.card, selectedId === item.id && styles.cardSelected]}
            onPress={() => handleSelect(item.id)}
          >
            <Text style={styles.icon}>{item.icon || '📁'}</Text>
            <Text style={styles.name}>{item.name}</Text>
            {item.count !== undefined && (
              <Text style={styles.count}>{item.count} chaînes</Text>
            )}
          </TouchableOpacity>
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, backgroundColor: '#121212' },
  title: { fontSize: 22, fontWeight: 'bold', color: '#fff', marginBottom: 16 },
  card: {
    flex: 1,
    margin: 8,
    backgroundColor: '#1e1e1e',
    borderRadius: 12,
    padding: 20,
    alignItems: 'center',
  },
  cardSelected: { backgroundColor: '#1a5fb4' },
  icon: { fontSize: 32, marginBottom: 8 },
  name: { color: '#fff', fontSize: 14, fontWeight: '500', textAlign: 'center' },
  count: { color: '#888', fontSize: 11, marginTop: 4 },
});

====================================================================
10. src/screens/SettingsScreen.tsx
====================================================================
import React from 'react';
import { View, Text, StyleSheet, Switch, TouchableOpacity, Alert } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';

export default function SettingsScreen() {
  const [autoPlay, setAutoPlay] = React.useState(false);
  const [epgEnabled, setEpgEnabled] = React.useState(true);
  const [darkMode, setDarkMode] = React.useState(true);

  const clearCache = async () => {
    Alert.alert('Confirmation', 'Vider le cache et les listes ?', [
      { text: 'Annuler', style: 'cancel' },
      {
        text: 'Vider',
        style: 'destructive',
        onPress: async () => {
          await AsyncStorage.clear();
          Alert.alert('Cache vidé ✅');
        },
      },
    ]);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>⚙️ Paramètres</Text>
      
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Lecture</Text>
        
        <View style={styles.row}>
          <Text style={styles.label}>Lecture automatique</Text>
          <Switch value={autoPlay} onValueChange={setAutoPlay} />
        </View>
        
        <View style={styles.row}>
          <Text style={styles.label}>Activer l'EPG</Text>
          <Switch value={epgEnabled} onValueChange={setEpgEnabled} />
        </View>
      </View>
      
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Apparence</Text>
        <View style={styles.row}>
          <Text style={styles.label}>Mode sombre</Text>
          <Switch value={darkMode} onValueChange={setDarkMode} />
        </View>
      </View>
      
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Stockage</Text>
        <TouchableOpacity style={styles.button} onPress={clearCache}>
          <Text style={styles.buttonText}>🗑️ Vider le cache et les données</Text>
        </TouchableOpacity>
      </View>
      
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>À propos</Text>
        <Text style={styles.version}>SuarezPlayer v1.0.0</Text>
        <Text style={styles.desc}>Lecteur IPTV libre et sans compte</Text>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, backgroundColor: '#121212' },
  title: { fontSize: 26, fontWeight: 'bold', color: '#fff', marginBottom: 24 },
  section: { marginBottom: 32 },
  sectionTitle: { fontSize: 18, fontWeight: '600', color: '#1a5fb4', marginBottom: 12 },
  row: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center', paddingVertical: 12, borderBottomWidth: 1, borderBottomColor: '#2a2a2a' },
  label: { color: '#fff', fontSize: 15 },
  button: { paddingVertical: 14, backgroundColor: '#2a2a2a', borderRadius: 8, alignItems: 'center', marginTop: 8 },
  buttonText: { color: '#ff6b6b', fontSize: 15 },
  version: { color: '#888', fontSize: 14 },
  desc: { color: '#666', fontSize: 12, marginTop: 4 },
});

====================================================================
11. src/screens/LoginScreen.tsx
====================================================================
import React, { useState } from 'react';
import { View, Text, TextInput, Button, StyleSheet, Alert } from 'react-native';
import { XtreamClient } from '../services/xtreamApi';
import AsyncStorage from '@react-native-async-storage/async-storage';

interface Props {
  onLogin: (client: XtreamClient) => void;
}

export default function LoginScreen({ onLogin }: Props) {
  const [serverUrl, setServerUrl] = useState('');
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [isLoading, setIsLoading] = useState(false);

  const saveCredentials = async () => {
    await AsyncStorage.setItem('xtream_server', serverUrl);
    await AsyncStorage.setItem('xtream_user', username);
    await AsyncStorage.setItem('xtream_pass', password);
  };

  const loadCredentials = async () => {
    const url = await AsyncStorage.getItem('xtream_server');
    const user = await AsyncStorage.getItem('xtream_user');
    const pass = await AsyncStorage.getItem('xtream_pass');
    if (url) setServerUrl(url);
    if (user) setUsername(user);
    if (pass) setPassword(pass);
  };

  React.useEffect(() => { loadCredentials(); }, []);

  const handleConnect = async () => {
    if (!serverUrl || !username || !password) {
      Alert.alert('Erreur', 'Remplissez tous les champs');
      return;
    }
    setIsLoading(true);
    try {
      const client = new XtreamClient({ serverUrl, username, password });
      await client.authenticate();
      await saveCredentials();
      onLogin(client);
    } catch (err) {
      Alert.alert('Erreur', 'Impossible de se connecter — vérifiez vos informations');
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>📺 SuarezPlayer</Text>
      <Text style={styles.subtitle}>Connexion au serveur IPTV</Text>
      
      <TextInput
        style={styles.input}
        placeholder="URL du serveur (ex: http://monserveur:8080)"
        value={serverUrl}
        onChangeText={setServerUrl}
        autoCapitalize="none"
        keyboardType="url"
      />
      <TextInput
        style={styles.input}
        placeholder="Nom d'utilisateur"
        value={username}
        onChangeText={setUsername}
        autoCapitalize="none"
      />
      <TextInput
        style={styles.input}
        placeholder="Mot de passe"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
        autoCapitalize="none"
      />
      
      <Button
        title={isLoading ? "Connexion..." : "Se connecter"}
        onPress={handleConnect}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 24, backgroundColor: '#121212', justifyContent: 'center' },
  title: { fontSize: 32, fontWeight: 'bold', color: '#fff', textAlign: 'center', marginBottom: 8 },
  subtitle: { fontSize: 16, color: '#888', textAlign: 'center', marginBottom: 32 },
  input: { backgroundColor: '#fff', padding: 12, borderRadius: 8, marginBottom: 16 },
});

====================================================================
12. src/components/SearchBar.tsx
====================================================================
import React from 'react';
import { View, TextInput, StyleSheet } from 'react-native';

interface Props {
  value: string;
  onChange: (text: string) => void;
  placeholder?: string;
}

export default function SearchBar({ value, onChange, placeholder = "Rechercher..." }: Props) {
  return (
    <View style={styles.container}>
      <TextInput
        style={styles.input}
        placeholder={placeholder}
        placeholderTextColor="#888"
        value={value}
        onChangeText={onChange}
        autoCapitalize="none"
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { marginBottom: 16 },
  input: {
    backgroundColor: '#2a2a2a',
    color: '#fff',
    padding: 12,
    borderRadius: 8,
    fontSize: 15,
  },
});

====================================================================
13. src/components/player/VideoPlayer.tsx
====================================================================
import React from 'react';
import { View, StyleSheet, Text } from 'react-native';
import { VideoView, useVideoPlayer } from 'expo-video';
import { usePlayerStore } from '../../store/playerStore';

interface Props {
  streamUrl: string | null;
}

export default function VideoPlayer({ streamUrl }: Props) {
  const { isPlaying, setPlaying } = usePlayerStore();
  
  const player = useVideoPlayer(
    streamUrl ? { uri: streamUrl } : null,
    (player) => {
      player.loop = false;
      player.play();
      setPlaying(true);
    }
  );

  if (!streamUrl) {
    return (
      <View style={styles.container}>
        <Text style={styles.placeholder}>Sélectionnez une chaîne</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <VideoView
        style={styles.video}
        player={player}
        allowsFullscreen
        allowsPictureInPicture
        nativeControls
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    width: '100%',
    height: 220,
    backgroundColor: '#000',
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 8,
    marginBottom: 12,
  },
  video: {
    width: '100%',
    height: '100%',
  },
  placeholder: {
    color: '#888',
    fontSize: 16,
  },
});

====================================================================
14. src/components/player/VideoControls.tsx
====================================================================
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

interface Props {
  isPlaying?: boolean;
  onPlayPause?: () => void;
  onFullscreen?: () => void;
  onSeekBack?: () => void;
  onSeekForward?: () => void;
  currentTime?: string;
  duration?: string;
}

export default function VideoControls({
  isPlaying,
  onPlayPause,
  onFullscreen,
  onSeekBack,
  onSeekForward,
  currentTime,
  duration,
}: Props) {
  return (
    <View style={styles.controls}>
      <View style={styles.progressBar}>
        <Text style={styles.time}>{currentTime || '0:00'}</Text>
        <Text style={styles.time}>{duration || '--:--'}</Text>
      </View>
      
      <View style={styles.buttons}>
        <TouchableOpacity style={styles.btn} onPress={onSeekBack}>
          <Text style={styles.btnText}>⏮</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.playBtn} onPress={onPlayPause}>
          <Text style={styles.playText}>{isPlaying ? '⏸' : '▶'}</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.btn} onPress={onSeekForward}>
          <Text style={styles.btnText}>⏭</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.btn} onPress={onFullscreen}>
          <Text style={styles.btnText}>⛶</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  controls: {
    position: 'absolute',
    bottom: 0,
    left: 0,
    right: 0,
    backgroundColor: 'rgba(0,0,0,0.7)',
    padding: 12,
  },
  progressBar: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 8,
  },
  time: { color: '#fff', fontSize: 12 },
  buttons: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    alignItems: 'center',
  },
  btn: { padding: 8 },
  btnText: { fontSize: 24, color: '#fff' },
  playBtn: { padding: 12 },
  playText: { fontSize: 32, color: '#fff' },
});

====================================================================
15. src/components/EPGGridView.tsx
====================================================================
import React, { useMemo } from 'react';
import { View, Text, ScrollView, StyleSheet } from 'react-native';
import { useEPGManagement } from '../hooks/useEPGManagement';

interface Props {
  selectedChannelId?: string;
}

export default function EPGGridView({ selectedChannelId }: Props) {
  const { channels, getProgramsForChannel } = useEPGManagement();
  const today = new Date();

  const hours = useMemo(() => {
    const start = new Date(today.setHours(0, 0, 0, 0));
    return Array.from({ length: 24 }, (_, i) => {
      const d = new Date(start);
      d.setHours(i);
      return d;
    });
  }, [today]);

  const getProgramPosition = (start: Date, end: Date) => {
    const startMinutes = start.getHours() * 60 + start.getMinutes();
    const endMinutes = end.getHours() * 60 + end.getMinutes();
    const totalWidth = 24 * 60;
    return {
      left: (startMinutes / totalWidth) * 100 + '%',
      width: ((endMinutes - startMinutes) / totalWidth) * 100 + '%',
    };
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>📅 Guide TV — {today.toLocaleDateString('fr-FR')}</Text>
      
      <ScrollView horizontal showsHorizontalScrollIndicator={false}>
        <View>
          <View style={styles.timeHeader}>
            <View style={styles.channelNameCol} />
            {hours.map(h => (
              <Text key={h.getHours()} style={styles.timeLabel}>
                {h.getHours()}h
              </Text>
            ))}
          </View>
          
          <ScrollView showsVerticalScrollIndicator={true}>
            {channels.length === 0 ? (
              <Text style={styles.empty}>Aucune donnée EPG — importez un fichier XMLTV</Text>
            ) : (
              channels.map(channel => (
                <View
                  key={channel.id}
                  style={[
                    styles.row,
                    selectedChannelId === channel.id && styles.rowSelected
                  ]}
                >
                  <View style={styles.channelNameCol}>
                    <Text style={styles.channelName} numberOfLines={1}>{channel.name}</Text>
                  </View>
                  
                  <View style={styles.programsRow}>
                    {getProgramsForChannel(channel.id, today).map(program => {
                      const pos = getProgramPosition(program.startTime, program.endTime);
                      return (
                        <View
                          key={program.id}
                          style={[styles.program, { left: pos.left, width: pos.width }]}
                        >
                          <Text style={styles.programTitle} numberOfLines={1}>
                            {program.title}
                          </Text>
                        </View>
                      );
                    })}
                  </View>
                </View>
              ))
            )}
          </ScrollView>
        </View>
      </ScrollView>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#121212', padding: 12 },
  title: { fontSize: 18, fontWeight: 'bold', color: '#fff', marginBottom: 12 },
  timeHeader: { flexDirection: 'row', borderBottomWidth: 1, borderBottomColor: '#333' },
  timeLabel: { width: 80, textAlign: 'center', color: '#888', fontSize: 12, paddingVertical: 6 },
  channelNameCol: { width: 120, paddingHorizontal: 8, justifyContent: 'center', borderRightWidth: 1, borderRightColor: '#333' },
  channelName: { color: '#fff', fontSize: 12 },
  row: { flexDirection: 'row', height: 50, borderBottomWidth: 1, borderBottomColor: '#2a2a2a' },
  rowSelected: { backgroundColor: '#1a2a44' },
  programsRow: { flex: 1, position: 'relative', height: 50 },
  program: { position: 'absolute', top: 4, bottom: 4, backgroundColor: '#1a5fb4', borderRadius: 4, paddingHorizontal: 4, justifyContent: 'center' },
  programTitle: { color: '#fff', fontSize: 10 },
  empty: { color: '#888', textAlign: 'center', marginTop: 40, padding: 20 },
});

====================================================================
16. src/hooks/useVideoPlayback.ts
====================================================================
import { useState } from 'react';

export function useVideoPlayback() {
  const [currentChannel, setCurrentChannel] = useState<any>(null);

  const playChannel = (channel: any) => {
    setCurrentChannel(channel);
    console.log('Lecture:', channel.name, channel.url);
  };

  return { playChannel, currentChannel };
}

====================================================================
17. src/hooks/useEPGManagement.ts
====================================================================
import { useQuery, useRealm } from '@realm/react';
import { EpgChannel, EpgProgram } from '../database/Schema';

export function useEPGManagement() {
  const realm = useRealm();
  const channels = useQuery(EpgChannel);
  const programs = useQuery(EpgProgram);

  const getProgramsForChannel = (channelId: string, date: Date = new Date()) => {
    const startOfDay = new Date(date.setHours(0, 0, 0, 0));
    const endOfDay = new Date(date.setHours(23, 59, 59, 999));
    
    return programs.filtered(
      'channelId == $0 AND startTime >= $1 AND startTime <= $2 SORT(startTime ASC)',
      channelId, startOfDay, endOfDay
    );
  };

  const getCurrentProgram = (channelId: string) => {
    const now = new Date();
    return programs.filtered(
      'channelId == $0 AND startTime <= $1 AND endTime > $1 LIMIT(1)',
      channelId, now
    )[0];
  };

  const importEpgData = (data: {
    channels: Array<{ id: string; name: string; logo?: string }>;
    programs: Array<{
      id: string; channelId: string; title: string; description?: string; startTime: Date; endTime: Date;
    }>;
  }) => {
    realm.write(() => {
      data.channels.forEach(c => {
        realm.create(EpgChannel, c, Realm.UpdateMode.Modified);
      });
      data.programs.forEach(p => {
        realm.create(EpgProgram, p, Realm.UpdateMode.Modified);
      });
    });
  };

  const clearOldEpg = () => {
    realm.write(() => {
      const oneWeekAgo = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000);
      const old = programs.filtered('endTime < $0', oneWeekAgo);
      realm.delete(old);
    });
  };

  return {
    channels,
    programs,
    getProgramsForChannel,
    getCurrentProgram,
    importEpgData,
    clearOldEpg,
  };
}

====================================================================
18. src/store/playerStore.ts
====================================================================
import { create } from 'zustand';

interface PlayerState {
  isPlaying: boolean;
  currentUrl: string | null;
  setPlaying: (v: boolean) => void;
  setUrl: (url: string | null) => void;
}

export const usePlayerStore = create<PlayerState>((set) => ({
  isPlaying: false,
  currentUrl: null,
  setPlaying: (v) => set({ isPlaying: v }),
  setUrl: (url) => set({ currentUrl: url })
}));

====================================================================
19. src/store/favoritesStore.ts
====================================================================
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

interface FavoriteItem {
  id: string;
  name: string;
  url: string;
  logo?: string;
  group?: string;
}

interface FavoriteState {
  items: FavoriteItem[];
  addFavorite: (channel: FavoriteItem) => void;
  removeFavorite: (id: string) => void;
  isFavorite: (id: string) => boolean;
  toggleFavorite: (channel: FavoriteItem) => void;
}

export const useFavoritesStore = create<FavoriteState>()(
  persist(
    (set, get) => ({
      items: [],
      
      addFavorite: (channel) => {
        set((state) => ({
          items: [...state.items, channel]
        }));
      },
      
      removeFavorite: (id) => {
        set((state) => ({
          items: state.items.filter(c => c.id !== id)
        }));
      },
      
      isFavorite: (id) => {
        return get().items.some(c => c.id === id);
      },
      
      toggleFavorite: (channel) => {
        const exists = get().isFavorite(channel.id);
        if (exists) {
          get().removeFavorite(channel.id);
        } else {
          get().addFavorite(channel);
        }
      },
    }),
    {
      name: 'favorites-storage',
      storage: createJSONStorage(() => AsyncStorage),
    }
  )
);

====================================================================
20. src/database/Schema.ts
====================================================================
import { Realm } from '@realm/react';

export class EpgChannel extends Realm.Object {
  declare id: string;
  declare name: string;
  declare logo?: string;

  static schema = {
    name: 'EpgChannel',
    primaryKey: 'id',
    properties: {
      id: 'string',
      name: 'string',
      logo: 'string?',
    },
  };
}

export class EpgProgram extends Realm.Object {
  declare id: string;
  declare channelId: string;
  declare title: string;
  declare description?: string;
  declare startTime: Date;
  declare endTime: Date;

  static schema = {
    name: 'EpgProgram',
    primaryKey: 'id',
    properties: {
      id: 'string',
      channelId: 'string',
      title: 'string',
      description: 'string?',
      startTime: 'date',
      endTime: 'date',
    },
  };
}

====================================================================
21. src/services/xtreamApi.ts
====================================================================
export interface XtreamConfig {
  serverUrl: string;
  username: string;
  password: string;
}

export interface XtreamCategory {
  category_id: string;
  category_name: string;
  parent_id: string | number;
}

export interface XtreamChannel {
  stream_id: number;
  name: string;
  now_playing?: { title?: string; end_time?: string };
  stream_icon?: string;
  group?: string;
}

export interface XtreamVod {
  stream_id: number;
  name: string;
  category_id: string;
  movie_image?: string;
  duration?: string;
  rating?: string;
}

export class XtreamClient {
  private baseUrl: string;
  private username: string;
  private password: string;

  constructor(config: XtreamConfig) {
    this.baseUrl = config.serverUrl;
    this.username = config.username;
    this.password = config.password;
  }

  private url(endpoint: string, params:Record<string, string> = {}) { const query = new URLSearchParams(params).toString(); return ${this.baseUrl}/player_api.php?username=${this.username}&password=${this.password}&action=${endpoint}${query ? '&' + query : ''}; }
async authenticate() { const res = await fetch(this.url('')); if (!res.ok) throw new Error('Authentification échouée'); return res.json(); }
async getLiveCategories(): Promise<XtreamCategory[]> { const res = await fetch(this.url('get_live_categories')); return res.json(); }
async getLiveStreams(categoryId?: string): Promise<XtreamChannel[]> { const params = categoryId ? { category_id: categoryId } : {}; const res = await fetch(this.url('get_live_streams', params)); return res.json(); }
async getVodCategories(): Promise<XtreamCategory[]> { const res = await fetch(this.url('get_vod_categories')); return res.json(); }
async getVodStreams(categoryId?: string): Promise<XtreamVod[]> { const params = categoryId ? { category_id: categoryId } : {}; const res = await fetch(this.url('get_vod_streams', params)); return res.json(); }
async getVodInfo(streamId: number) { const res = await fetch(this.url('get_vod_info', { stream_id: String(streamId) })); return res.json(); }
getStreamUrl(streamId: number, ext = 'm3u8') { return ${this.baseUrl}/live/${this.username}/${this.password}/${streamId}.${ext}; }
getVodUrl(streamId: number) { return ${this.baseUrl}/movie/${this.username}/${this.password}/${streamId}.mp4; } }
export interface M3UItem { name: string; url: string; logo?: string; group?: string; id?: string; }
export async function fetchM3U(url: string): Promise<M3UItem[]> { try { const response = await fetch(url); if (!response.ok) throw new Error('Erreur réseau'); const text = await response.text(); return parseM3U(text); } catch (error) { console.error('Erreur chargement M3U:', error); throw error; } }
export function parseM3U(content: string): M3UItem[] { const lines = content.split('\n'); const items: M3UItem[] = [];
let tempItem: Partial<M3UItem> = {};
for (let i = 0; i < lines.length; i++) { const line = lines[i].trim();
if (line.startsWith('#EXTINF:')) {
  const nameMatch = line.match(/,(.*)$/);
  const logoMatch = line.match(/tvg-logo="([^"]*)"/);
  const groupMatch = line.match(/tvg-group="([^"]*)"/);
  
  tempItem.name = nameMatch ? nameMatch[1].trim() : 'Canal sans nom';
  tempItem.logo = logoMatch ? logoMatch[1] : undefined;
  tempItem.group = groupMatch ? groupMatch[1] : undefined;
} 
else if (line && !line.startsWith('#') && line.startsWith('http')) {
  tempItem.url = line;
  if (tempItem.name) {
    items.push({
      name: tempItem.name,
      url: tempItem.url,
      logo: tempItem.logo,
      group: tempItem.group,
    });
    tempItem = {};
  }
}
}
return items; }
export interface EpgProgramme { channel: string; start: Date; stop: Date; title: string; subTitle?: string; description?: string; icon?: string; }
function parseTime(timeStr: string): Date { const year = parseInt(timeStr.slice(0, 4)); const month = parseInt(timeStr.slice(4, 6)) - 1; const day = parseInt(timeStr.slice(6, 8)); const hour = parseInt(timeStr.slice(8, 10)); const minute = parseInt(timeStr.slice(10, 12)); const second = parseInt(timeStr.slice(12, 14)) || 0; return new Date(year, month, day, hour, minute, second); }
export function parseXMLTV(xml: string) { const programs: EpgProgramme[] = [];
const programmeRegex = /<programme[^>]channel="([^"]+)"[^>]start="([^"]+)"[^>]stop="([^"]+)"[^>]>([\s\S]?)</programme>/gi; const titleRegex = /<title[^>]>([\s\S]?)</title>/i; const subTitleRegex = /<sub-title[^>]>([\s\S]?)</sub-title>/i; const descRegex = /<desc[^>]>([\s\S]*?)</desc>/i; const iconRegex = /<icon[^>]src="([^"]+)"[^>]/>/i;
let match; while ((match = programmeRegex.exec(xml)) !== null) { const [_, channelId, startStr, stopStr, content] = match;
const titleMatch = content.match(titleRegex);
const subTitleMatch = content.match(subTitleRegex);
const descMatch = content.match(descRegex);
const iconMatch = content.match(iconRegex);

try {
  programs.push({
    channel: channelId,
    start: parseTime(startStr),
    stop: parseTime(stopStr),
    title: titleMatch ? titleMatch[1].trim() : 'Sans titre',
    subTitle: subTitleMatch ? subTitleMatch[1].trim() : undefined,
    description: descMatch ? descMatch[1].trim() : undefined,
    icon: iconMatch ? iconMatch[1] : undefined,
  });
} catch (e) {
  console.warn('Erreur parsing programme:', e);
}
}
return programs; }
export async function fetchEPG(url: string) { const res = await fetch(url); if (!res.ok) throw new Error('Impossible de charger l'EPG'); const xml = await res.text(); return parseXMLTV(xml);  
node_modules/
dist/
.expo/
*.log
.env
 npm install
npx expo start
